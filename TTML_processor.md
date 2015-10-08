# TTML processing

The current processing takes place between the TTML editor (who doesn't want to care about further packaging) and the MP4/MPEG-DASH muxer (which don't have to know about TTML further than 14496-30 or profile-specific equivalent).

### What it does

- segmenting (split TTML content samples at given timepoints)
- timebase and duration conversion
