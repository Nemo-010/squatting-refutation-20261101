Part of the refutation sweep (#1).

**Claim under test.** Restreaming is found by RTMP/SRT/HLS listeners, per-socket byte growth, HLS request-pattern analysis in the web log, `ffmpeg` command lines containing `rtmp://`/`srt://`, and the fan-out / flatness / cost-coverage ratios (`research/streaming-IPTV-and-bandwidth-resale.md` §5–§6).

**P13 / tool substitution.** The chain is "an encoder publishes to an ingest". `ffmpeg` renamed, OBS, gstreamer, VLC or `nginx-rtmp` itself all do the same job. The corpus's own documented campaign used `ffmpeg`, and the grep for `rtmp://` in `cmdline` is one `exec -a` or one `-i -` away from nothing.

**Input from a pipe.** `ffmpeg -i -` reads from stdin; a wrapper opens the source and pipes it in. No URL in the command line, no `rtmp://` string.

**Push to a CDN and fan out there.** One outbound flow from the host, N viewers at the CDN. The HLS request-pattern analysis reads the *origin* access log, and the origin is not the CDN: the box makes one connection.

**WebRTC instead of RTMP.** UDP, dynamic ports, no 1935 listener, and the media server is a legitimate one (`mediamtx`).

**P17 — stop being a metronome.** `-re` paces at exactly real time, which is the flatness signal. Variable-bitrate encoding and bursty segment delivery remove it; the viewer experience is slightly worse and the detector is blind.

**P18 — add ingress.** Fan-out is `out/in` on one path. A download running alongside the restream — an update, a backup pull, a second restream — moves the ratio toward 1.

**Verdict: medium.** The application counters (`nclients`, `bytes_out` on one path) are the strongest item, and they only exist if the operator's own media server is the origin — which is precisely the deployment a reseller avoids. The log-based checks measure the last hop the reseller controls.
