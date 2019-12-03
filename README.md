little-media-box
==========

> Convenient atomicized classes for representing digital multimedia assets
> in distributed Node.js DSP pipelines.

## Installation

> WIP

## Status

> **Development/Testing/Documentation**

## Usage

```js
const Media = require('little-media-box')

async function main() {
  const src = await new Media.Source('http://distribution.bbb3d.renderfarming.net/video/mp4/bbb_sunflower_1080p_60fps_normal.mp4')
  // const src = await new Media.Source('test.mp4')

  /* Derive tracks from the source */
  const videoTrack = new Media.VideoTrack(src) // video @ track 1/3
  const mainAudioTrack = new Media.AudioTrack(src) // stereo audio @ track 2/3
  const secAudioTrack = new Media.AudioTrack(src, 2) // 5.1 audio @ track 3/3

  /* Create a MediaPackage */
  const pack = new Media.Package([videoTrack, mainAudioTrack, secAudioTrack])
  console.log(pack)
}

main()
```

Produces the following output:

```sh
Package {
  tracks: [
    VideoTrack {
      source: [Source],
      properties: [Object],
      mediaType: 'video',
      primary: true,
      valid: true,
      smpteTimecode: [Timecode]
    },
    AudioTrack {
      source: [Source],
      properties: [Object],
      mediaType: 'audio',
      primary: true,
      valid: true
    },
    AudioTrack {
      source: [Source],
      properties: [Object],
      mediaType: 'audio',
      primary: true,
      valid: true
    }],
  opts: {}
}
```

## API

> WIP

### `const delivery = new Delivery(sources[, options])`

Creates and returns a new `Delivery` object, comprised of the provided
`sources`, and modified with the optionally-specified `options`. This object
should contain every media asset necessary to designate the content delivery
complete.

### `const package = new Package(tracks[, options])`

Creates and returns a new `Package` object, comprised of the provided
`tracks`, and modified with the optionally-specified `options`. This object
should contain every media track necessary to ingest, distribute, and view
one complete piece of media content.

#### `await package.mux(opts = {})`

Muxes each `Track` contained within the `Package`, producing a single Matroska
file.

##### `opts.outputUrl`

The path of the output file.

### `const source = new Source(uri[, options])`

Creates and returns a new `Source` object from `uri`. `uri` may be one local
file path, or an accessible HTTP/HTTPS location, in which case the file at that
location will be pulled down and read into a `ReadableStream` for consumption
by `ffprobe`.

#### `await source.demux()`

Demuxes each item in `source.properties.streams` to its own individual Matroska
container. Default output URI format:

```js
`${stream.index}_${stream.codec_type}.mkv`
```

#### `source.properties`

A read-only accessor for the media properties and metadata obtained from
`ffprobe`.

##### `source.properties.streams`

A read-only accessor for array of individual media bytestreams contained in the
file accessed from `source.uri`.

##### `source.properties.format`

A read-only accessor for the container-level metadata in the file accessed from
`source.uri`.

### `const track = new Track(source, index = 0)`

Creates and returns a new `Track` object, which represents the individual media
bytestream obtained from the provided `source.uri` at the given `index` number.

#### `const audioTrack = new AudioTrack(source, index=1)`

Creates and returns a new `AudioTrack` object, which extends `Track`. Throws an
error if the individual media bytestream obtained from the provided `source.uri`
at the given `index` number is not a valid audio bytestream.

#### `const subtitleTrack = new SubtitleTrack(source, index=2)`

Creates and returns a new `SubtitleTrack` object, which extends `Track`. Throws an
error if the individual media bytestream obtained from the provided `source.uri`
at the given `index` number is not a valid subtitle bytestream.

#### `const videoTrack = new VideoTrack(source, index=0)`

Creates and returns a new `VideoTrack` object, which extends `Track`. Throws an
error if the individual media bytestream obtained from the provided `source.uri`
at the given `index` number is not a valid video bytestream.

##### `videoTrack.parseSmpte()`

Generates and returns a SMPTE Timecode object from the track's media properties.
Also stores the object at `videoTrack.smpteTimecode`. Film, PAL, NTSC and ATSC
are supported, in both regular and drop-frame formats. When a frame rate which
doesn't conform with SMPTE is present, an attempt will be made to create a 
best-guess SMPTE timecode, and `{ valid: false }` will be set within the
`smpteTimecode` object.

##### `videoTrack.getSmpteTimecode()`

Generates and returns a string of the track's
[SMPTE Timecode](https://en.wikipedia.org/wiki/SMPTE_timecode).

##### `videoTrack.getImmersive()`

A convenience function, which returns an object containing relevant information
about the track's immersive media qualities.

```js
videoTrack.getImmersive()
// Returns: { projection: 'equirectangular', stereoscopy: 'over-under' }
```

##### `videoTrack.primary`

Indicates whether the `videoTrack` is the *primary* video track of its given `Source`.

*Primary* `videoTrack`s are hoisted to the top of `Package.tracks[]` when added
to a new `Package`.

##### `videoTrack.valid`

A boolean indicating whether the `videoTrack` contains a valid video bytestream.

When `false`, a new `Error` is thrown by the `videoTrack` constructor.

## License

MIT
