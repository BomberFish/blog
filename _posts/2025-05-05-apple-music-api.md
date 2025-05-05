---
tags: Writeup
description: While working on a tool which transfers playlists between music streaming services, I had to reverse-engineer Apple Music's API. This article goes over some of the endpoints the API has to offer.
---
# Notes on reverse-engineering the Apple Music API
I've recently been working on Transcribe, a tool which transfers playlists between music streaming services directly inside your browser ([Demo video](https://youtu.be/MshSHxytkdQ)). To add support for Apple Music, I had to reverse-engineer its API. Now, you may be wondering:
## Why not just use MusicKit?
For anyone who doesn't know, MusicKit (JS) is Apple Music's **official API**. The issue with MusicKit is that using it requires an active Apple Developer Program membership. This would require me to pay Apple almost $150 every year, which is money I do not have. So, I had to resort to a more legally dubious method.

Now without any further ado, let's get into the cool stuff! 

> Please note that this is far from complete and should not be considered complete documentation. It might be updated in the future. Also, please hire me Apple :3

## How I did it
This is very simple. I simply opened the [Apple Music Web Client](https://beta.music.apple.com) in my browser, opened the Network tab in Chrome's DevTools, and watched any fetch requests it made.

The base URL for all requests is `https://amp-api.music.apple.com`.

## Authentication
The API seems to need two pieces of information to accept requests: a user token (via the `Authentication` header) and some cookies.

As for the cookies, I haven't looked into what parts of the cookie the API actually requires to be there, but pasting everything sent in the `Cookie` header works fine.

## Common data types

Here are a selection of TypeScript types from Transcribe. I'll refer to them later in example responses.

```ts
/**
 * Represents a generic reference to an Apple Music resource.
 */
interface AMResourceReference {
  id: string;
  type: AMResourceType;
  href: string;
}

/**
 * Represents a relationship link to other resources.
 */
interface AMRelationship<T extends AMResourceReference> {
  href: string;
  data: T[];
  meta?: { // Optional meta information, seen for tracks
    total: number;
  };
}

/**
 * Container for included resource objects, keyed by type, then ID.
 */
interface AMResourcesContainer {
  'library-playlists'?: { [id: string]: AMLibraryPlaylist };
  'playlists'?: { [id: string]: AMCatalogPlaylist };
  'library-songs'?: { [id: string]: AMLibrarySong };
  'songs'?: { [id: string]: AMCatalogSong };
  'apple-curators'?: { [id: string]: AMAppleCurator };
  'artists'?: { [id: string]: AMArtist };
  'albums'?: { [id: string]: AMAlbum };
  'music-videos'?: { [id: string]: AMMusicVideo };
  'stations'?: { [id: string]: AMStation };
  'uploaded-videos'?: { [id: string]: AMUploadedVideo };\
}
```

## Getting stuff from your Library
### Recently Added
**Endpoint:** `GET /v1/me/library/recently-added`

### All Playlists
**Endpoint:** `GET /v1/me/library/playlists`

You can limit the number of playlists returned with **the `limit` parameter.**
**What the response would look like:**
```ts
[
  {
	tracks: AMRelationship<AMResourceReference>,
	catalog?: AMRelationship<AMResourceReference>,
  }
]
```
### Songs in a specific playlist
**Endpoint:** `GET /v1/me/library/playlists/:id`

**What the response would look like:**
```ts
[
  {
	data: AMResourceReference[],
	resources?: AMResourcesContainer,
  }
]
```
## Managing your Library
### Creating a Playlist
**Endpoint:** `POST /v1/me/playlists`
**What your body should look like:**
```ts
{
  attributes: {
    name: string;
	description: string;
	isPublic: boolean;
  },
  relationships: {
  	tracks: {
	  data: [
	  	id: string,
		type: "songs"|"library-songs"|"music-videos",
	  ]
	}
  }
}
```

**What the response would look like:**
```ts
[
  {
  	id: string;
  	type: "library-playlists",
  	attributes: any,
  }
]
```
## Other
### Search
**Endpoint:** `GET /v1/catalog/ca/search`
This one's a bit tricky. I haven't quite cracked how it handles parameters, but here's what parameters I use in Transcribe to search for only songs:

```js
let params = {
    "art[music-videos:url]": "c",
    "art[url]": "f",
    extend: "artistUrl",
    "fields[albums]":
        "artistName,artistUrl,artwork,contentRating,editorialArtwork,editorialNotes,name,playParams,releaseDate,url,trackCount",
    "fields[artists]": "url,name,artwork",
    "format[resources]": "map",
    "include[albums]": "artists",
    "include[songs]": "artists",
    l: "en-CA",
    limit: limit,
    "omit[resource]": "autos",
    platform: "web",
    "relate[albums]": "artists",
    "relate[songs]": "albums",
    term: query,
    types: "songs",
    with: "lyricHighlights,lyrics,serverBubbles,subtitles",
};
```

**Relevant types for the response:**
```ts
/**
 * The top-level structure of the Apple Music API search response.
 */
interface AMSearchResponse {
    results: AMSearchResults;
    resources?: AMResourcesContainer; // Included full resources (optional based on request/results)
    meta?: AMSearchMeta; // Optional meta information
}

/**
 * Represents the 'results' part of the search response, containing different groups.
 * Keys are typically the `groupId` values.
 */
interface AMSearchResults {
    top?: AMTopSearchResultGroup;
    artist?: AMSearchResultGroup<AMResourceReference>;
    album?: AMSearchResultGroup<AMResourceReference>;
    song?: AMSearchResultGroup<AMResourceReference>;
    playlist?: AMSearchResultGroup<AMResourceReference>;
    'radio_episode'?: AMSearchResultGroup<AMResourceReference>; // Note: uses station type reference
    station?: AMSearchResultGroup<AMResourceReference>;
    'music_video'?: AMSearchResultGroup<AMResourceReference>;
    'video_extra'?: AMSearchResultGroup<AMResourceReference>; // Note: uses uploaded-video type reference
    [key: string]: AMTopSearchResultGroup | AMSearchResultGroup<any> | undefined; // Index signature for flexibility
}

/**
 * Represents the 'top' results group which contains mixed types.
 */
interface AMTopSearchResultGroup {
    // No href/next at this level
    data: AMResourceReference[];
    name: string; // e.g., "Top Results"
    groupId: 'top';
}

/**
 * Represents a group of search results for a specific type (e.g., artists, albums).
 */
interface AMSearchResultGroup<T extends AMResourceReference> {
    href?: string; // Link to refine search for this type
    next?: string; // Link to next page of results for this type
    data: T[]; // Array of references for this type
    name: string; // Display name (e.g., "Artists")
    groupId: string; // Internal group ID (e.g., "artist")
}

/**
 * Represents the 'meta' part of the search response.
 */
interface AMSearchMeta {
    results: {
        order: string[]; // Array of groupIds in display order
        rawOrder: string[]; // Array of groupIds potentially including types with no results
    };
    metrics?: { // Optional metrics data
        dataSetId: string;
    };
}

```