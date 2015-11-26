# TTML in MP4 and MPEG-DASH guidelines

### About

Authors:
 - Romain Bouqueau (romain.bouqueau@gpac-licensing.com)
 - Cyril Concolato (cyril.concolato@telecom-paristech.fr)

Contributors:
 - Nigel Megitt (nigel.megitt@bbc.co.uk)
 - Andreas Tai (tai@irt.de)
 - Frans de Jong (dejong@ebu.ch)

### Introduction

This document describes different workflows for the delivery of TTML content in MP4 and MPEG-DASH. It tries to provide hints on how to build such workflows based on existing tools. Its goal is to drive the development of TTML tools such that maximum interoperability is achieved.

### History and profiles for TTML: EBU work, HbbTV 2.0, IMSC

About the technologies used in this article:
 - [MP4](https://en.wikipedia.org/wiki/MPEG-4_Part_14) is a container format standardized by MPEG.
 - [MPEG-DASH](http://standards.iso.org/ittf/PubliclyAvailableStandards/c065274_ISO_IEC_23009-1_2014.zip) is an adaptive bitrate (ABR) streaming standard allowing delivery of content using conventional HTTP web servers also standardized by MPEG.
 - [TTML](http://www.w3.org/TR/ttml1/) (Timed Text Markup Language) is a subtitling format designed by W3C.
 - [EBU-TT-D](https://tech.ebu.ch/publications/tech3380) is a profile of TTML designed for live and on demand distribution of subtitles over IP based networks. EBU-TT-D restricts TTML. EBU-TT-D is designed by the EBU.
 - [HbbTV 2.0](https://www.hbbtv.org/pages/about_hbbtv/HbbTV_specification_2_0.pdf) is a standard for hybrid digital TV delivery. HbbTV 2.0 mandates the implementation of EBU-TT-D, MP4 and MPEG-DASH.
 - [IMSC](http://www.w3.org/TR/ttml-imsc1/) is a pair of TTML profiles, one for text and one for images designed for subtitles and captions. The IMSC Text profile is a superset of EBU-TT-D.

### Overview of the workflow
Generally speaking, we can assume that TTML workflows follow the architecture provided by the following image:
![Image of Workflow](/TTMLWorkflow.png)

In this workflow, the MP4 packager and DASH packager could be the same tool, as it is the case with MP4Box. Similarly, the DASH Access Engine and the MP4 Parser and the TTML Renderer could be the same tool, or separate tools such as respectively DASH.js, MP4Box.js and a TTML to HTML rendering tool.

#### Producing TTML content over MP4 and DASH

Given this workflow, there are several options to produce, package and deliver TTML content over MP4 and DASH. All options have in commom that they try to minimize the quantity of downloaded data during the streaming session: this means avoiding downloading the same TTML content multiple times; and at the same time not requiring the download of the whole TTML content to start the session (especially relevant for live applications). Packaging the TTML content of the entire session as a single DASH segment is indeed not optimal. Packaging of the TTML requires the content to be spread over multiple DASH segments. This can be useful for seeking or for inserting ads between segments. 

DASH segments are typically of constant duration and aligned across audio and video representations. This is not a strict requirement though. Since TTML content does not usually have a fixed frame rate, segmentation of TTML content may lead to either variable duration segments or to data duplication across segments. Such duplication should be avoided and limited, possibly to the last sample of a segment containing some data that is present in the first sample of the next segment.

#### Need for TTML post-processing

In above workflow it may be difficult for tools that have only simple TTML capabilities, to process a TTML document for the purpose of creating small, self contained, non overlapping TTML documents (sometimes called intermediate synchronic document, ISD). The example below shows a TTML document with successive `p` elements overlapping in time. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tt:tt  xmlns:ttp="http://www.w3.org/ns/ttml#parameter" xmlns:tts="http://www.w3.org/ns/ttml#styling" xmlns:tt="http://www.w3.org/ns/ttml" ttp:timeBase="media" xml:lang="de" ttp:cellResolution="50 30">
	<tt:head>
		<tt:styling>
			<tt:style xml:id="textWhite" tts:color="#ffffff" tts:backgroundColor="#000000" tts:fontSize="160%" tts:fontFamily="monospace" />
			<tt:style xml:id="paragraphAlign" tts:textAlign="left" />
		</tt:styling>
		<tt:layout>
			<tt:region xml:id="top" tts:origin="10% 10%" tts:extent="80% 80%" tts:displayAlign="before"/>
			<tt:region xml:id="bottom" tts:origin="10% 10%" tts:extent="80% 80%" tts:displayAlign="after" />	
		</tt:layout>
	</tt:head>
	<tt:body>
		<tt:div>
			<tt:p xml:id="subtitle1" region="top" begin="00:00:00.000" end="00:00:04.000" style="paragraphAlign">
				<tt:span style="textWhite">Title at the top.</tt:span>
			</tt:p>
			<tt:p xml:id="subtitle2" region="bottom" begin="00:00:02.000" end="00:00:06.000" style="paragraphAlign">
				<tt:span style="textWhite">Text at the bottom.</tt:span>
			</tt:p>
		</tt:div>
	</tt:body>
</tt:tt>
```

Some workflows may decide that the TTML Authoring tool will post-process the TTML content to produce those ISD with a fine granularity to support the smallest segment duration and take care of timebase conversions. Such a post-processing tool would make the task of tools down the chain easier. Other workflows may decide to leave the segmentation to tools down the chain like the DASH packager because the segment duration is only known at that level in the workflow. Yet other workflows may use tools in-between to make the TTML authoring DASH-unuware and the DASH processing TTML-unaware. Depending on the design choice, the interface between the tools in the workflow will not be the same.

#### Interface between MP4 Parser and TTML Renderer
The MP4 standard assumes that only one sample at a time is active. This means that the MP4 parser will deliver one TTML document at a time to the TTML renderer and will assume that the previous TTML document will be replaced by the new one, and that it will be used for a given duration. This standard behavior thus constrains the upper part of the workflow, in the sense that samples cannot overlap and therefore the contained TTML document should not overlap. This improves interoperability by reducing the number of choices left.

Some optimizations at the MP4 level allow for the MP4 Parser to indicate that a new TTML document is the same as the previous one i.e. using the ```sample_has_redundancy``` field of the [```sdtp``` box](https://github.com/gpac/mp4box.js/blob/9f0bf463a979aa795e83a488360ed9db0fbf1329/src/parsing/sdtp.js). In this case the new document only extends the duration of the previous one. This can be useful when a TTML document has been duplicated between the last sample of a segment and the first sample of the next segment.

#### Interface between TTML Authoring Tool and MP4 Packager
There are several possibilities here. To achieve interoperability, workflow designers have to choose a strategy and make sure the tools are the right ones. This depends on the TTML Authoring tool. This tool may produce:
 - A single TTML document valid for the entire streaming session. If so, either the MP4 packager will have to split the TTML document into multiple samples, or the DASH packager will have to split the sample into multiple samples and segments to avoid unnecessary downloads. This task can be complex for general TTML documents, but it can be simpler for some profiles, such as EBU-TT-D. Hence, the workflow architecture may differ depending on the type of TTML documents.
 - Multiple non-overlapping TTML documents. If the TTML authoring tool is aware of the target DASH segment duration, it should ideally provide one TTML document per segment. If the TTML authoring tool is not aware of the DASH delivery parameters, it should try to produce the TTML documents with the smallest duration that cannot be further split. 

### Conclusion

If you have any feedback, remarks and questions. Please free to contact us directly or via [our github project page](https://github.com/rbouqueau/TTML_in_MP4_DASH_statement). Thank you.
