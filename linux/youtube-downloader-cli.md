# Youtube Downloader CLI

We will use Docker to keep our system clean and isolated.

## 1. Create a Dockerfile

```Dockerfile
FROM python:3.11-slim

# Install ffmpeg dependencies
RUN apt-get update && apt-get install -y \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# Install yt-dlp
RUN pip install --no-cache-dir yt-dlp

# Set the working directory
WORKDIR /downloads
```

## 2. Build the Docker Image

```bash
docker build -t yt-dlp-local .
```

## List Available Video and Audio Formats

To list all available formats for a given YouTube video, run the following
command. Replace `<video_url>` with the actual YouTube URL.

```bash
docker run --rm -v "$(pwd)":/downloads yt-dlp-local yt-dlp --list-formats "<video_url>"
```

## Download a Video with 720p Resolution

To download a video in 720p resolution, use the following command. Replace
`<video_url>` with the actual YouTube URL.

```bash
docker run --rm -v "$(pwd)":/downloads yt-dlp-local yt-dlp -f "bestvideo[height=720][ext=mp4]+bestaudio[ext=m4a]" "<video_url>"
```

This command will automatically join video and audio file

## Manually Join Audio and Video

To manually combine audio and video files, use the following command. Replace
`video.mp4` and `audio.m4a` with the actual file names, and `output.mp4` with
the desired output file name.

```bash
docker run --rm -v "$(pwd)":/downloads yt-dlp-local ffmpeg -i video.mp4 -i audio.m4a -c:v copy -c:a aac -strict experimental output.mp4
```

## Download Audio and Convert to MP3 Format

To download the audio from a video and convert it to MP3 format, use the
following commands. Replace `<video_url>` with the actual YouTube URL and
`audio.m4a` with the actual audio file name.

```bash
docker run --rm -v "$(pwd)":/downloads yt-dlp-local yt-dlp -f bestaudio[ext=m4a] "<video_url>"
docker run --rm -v "$(pwd)":/downloads yt-dlp-local ffmpeg -i audio.m4a -acodec libmp3lame -ab 192k audio.mp3
```
