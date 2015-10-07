# TTML in MP4 and MPEG-DASH guidelines

### Introduction

We'd like to provide clear guidelines for implementors who need to support TTML in their MP4 and MPEG-DASH workflows. These implementors can be:
 - TTML creators who intend to package their own content.
 - Packagers of third-party TTML content, MP4 and MPEG-DASH packager implementors.

### History and profiles for TTML: EBU work, HbbTV 2.0, ISMC

About the technologies used in this article:
 - MP4 is a container format standardized by MPEG.
 - MPEG-DASH is an adaptive bitrate (ABR) streaming standard allowing delivery of content using conventional HTTP web servers also standardized by MPEG.
 - TTML (Timed Text Markup Language) is a subtitling format designed by W3C.
 - EBU-TTD is a profile of TTML designed for live distribution. EBU-TTD restricts TTML. EBU-TTD is designed by EBU.
 - HbbTV 2.0 is a standard for hybrid digital TV delivery. HbbTV 2.0 mandates the use of EBU-TTD, MP4 and MPEG-DASH.

### Overview of the workflow
Schema:

```TTML creator => MP4 muxer => DASH packaging => ... delivery ... => DASH client => MP4 demuxer (extracts the TTML samples/documents) => TTML samples```

In this article, we only focus on both ends of the workflow about MP4 and TTML i.e. storing/extracting TTML documents in/from MP4 files. We are not concerned about the DASH packaging, or playback.

### Technical part - general guidelines

#### 1) TTML

Timings information can be present at several locations:
 - at the TTML document level and
 - at the MP4 level.

To prepare your content, you need to have access to the timing information one way or the other:
   - Case A: if your input is a raw TTML document, you need to parse the TTML document to find the timings of the samples. In any case you don't need to modify the timings of TTML document.
   - Case B: if your input is a MP4 document, you already have access to timing information.

The parsing of TTML may seem complex. Fortunately finding which profile of TTML you are implementing as it would allow to only parse a subset of TTML (such as EBU-TTD as described in section TTML above).
 
#### 2) MP4

The MP4 container guarantees that the TTML decoder will only be fed with one "active" document at a time.

#### 3) MPEG-DASH

If you plan to use MPEG-DASH to deliver your content, please note that your content needs to be segmented. Each segment contains a short interval of time of content.

Most of the time, MPEG-DASH packagers also have to prepare audio and video content. Segments from the different components (audio, video, subtitles, graphics, etc.) are often temporarily aligned (although this is not a standard requirement).

About timing overlaps: In a TTML document, several subtitle element may have overlapping timestamps. For example: EXAMPLE

When a subtitle element runs along several MPEG-DASH segments, the DASH packager needs to detect the timestamps from the TTML document to compose documents that contain all the content needed along the segment duration. For example: EXAMPLE

Optimization: if some subtitle elements runs across several MPEG DASH segments (e.g. 1. segments are short or 2. the TTML document couldn't be prepared in an optimized way), then you can use the redundancy flag at the MP4 container level when DASHing the content. It allows the TTML processor to know that the data doesn't need to be processed again. In this case, the MP4 timings shall be trusted (as ever). It allows optimized MP4 demuxers such as mp4box.js to avoid downloading some content that was already downloaded.

### Implementation advices summary
 1) If you author TTML, you should prepare your content so that it fits within your MPEG-DASH segment duration. Each TTML document should last no longer than a MPEG-DASH segment and there shall be no overlap.
 2) In any case you don't need to modify the timings of TTML document.
 3) If the TTML is already authored, you need to split a current document. This can occur to deal with overlapping when muxing in MP4 or when packaging for MPEG-DASH. When DASH-packaging, you can signal redundancy using the MP4 redundancy flag.

### Presentation of partners and relay on all blogs with cross-links
 - GPAC Licensing
 - Telecom ParisTech
 - BBC R&D
 - IRT
 - EBU

### References
 - [1] TTML
 - [2] EBU-TTD: https://tech.ebu.ch/docs/tech/tech3380.pdf
 - [3] HbbTV 2.0
 - [4] ISOBMF/MP4
 - [5] ISO14496-30
 - [6] EBU-TTD in MP4 https://tech.ebu.ch/docs/tech/tech3381.pdf
