# ffmpeg

## Cut video

```
ffmpeg -i "officiel - Live du lancement de la session.mp4" -ss 5:43:39 -c copy -t 262 -an output.mp4
```

- `ss <hh:mm:ss>` start timestamp
- `-t <seconds>` duration of the clip to cut
- `-an` to remove audio
