# AstrBot Text2Image Service

[中文](README_zh-CN.md) | English | [日本語](README_ja.md)

## Features

A simple web service that converts HTML/templates to images, with image lifecycle management support.

## Environment Variables

- `PORT`: Service port, default is 8999
- `IMAGE_LIFETIME_HOURS`: Image lifetime in hours, default is 24 hours. Images older than this will be automatically cleaned up
- `RATE_LIMIT_MAX_REQUESTS`: Maximum requests per rate-limit window, default 0 disables rate limiting
- `RATE_LIMIT_WINDOW_SECONDS`: Rate-limit window in seconds, default 0 disables rate limiting
- `DEFAULT_IMAGE_TYPE`: Default screenshot type, supports `png` / `jpeg`, default `png`
- `DEFAULT_IMAGE_QUALITY`: Default JPEG quality from `0-100`, only applies when the final output type is `jpeg`
- `DEFAULT_DEVICE_SCALE_FACTOR_LEVEL`: Default device pixel ratio level, supports `normal` / `high` / `ultra`, default `normal`

## Docker

Build the image:

```bash
docker build -t astrbot-t2i-service:local .
```

Run the container:

```bash
docker run --rm -p 8999:8999 \
  -e IMAGE_LIFETIME_HOURS=24 \
  -e DEFAULT_IMAGE_TYPE=jpeg \
  -e DEFAULT_IMAGE_QUALITY=90 \
  -e DEFAULT_DEVICE_SCALE_FACTOR_LEVEL=high \
  -v astrbot-t2i-data:/app/data \
  astrbot-t2i-service:local
```

## API Endpoints

### GET /health

Health check endpoint.

### POST /text2img/generate

Convert HTML to image

> Choose either html or tmpl. Provide tmpl and tmpldata together.

- `str` html: HTML text
- `str` tmpl: Jinja2 HTML template
- `dict` tmpldata: Jinja2 template data
- `bool` json: Whether to return JSON format (returns an id)
- `dict` `optional` options
  - timeout (float, optional): Screenshot timeout.
  - type (Literal["jpeg", "png"], optional): Screenshot image type.
  - quality (int, optional): Screenshot quality, only applicable to JPEG format.
  - omit_background (bool, optional): Whether to hide the default white background, allowing transparent screenshots (PNG only).
  - full_page (bool, optional): Whether to capture the entire page instead of just the viewport, default is True.
  - clip (FloatRect, optional): Area to clip after screenshot, xy is the starting point.
  - animations: (Literal["allow", "disabled"], optional): Whether to allow CSS animations.
  - caret: (Literal["hide", "initial"], optional): When set to `hide`, the text caret will be hidden during screenshot, default is `hide`.
  - scale: (Literal["css", "device"], optional): Page scaling settings. When set to `css`, device resolution maps 1:1 with CSS pixels, making screenshots smaller on high-DPI screens. When set to `device`, scales according to device screen scaling or the device_scale_factor parameter in the current Playwright Page/Context.
  - viewport_width (int, optional): Custom viewport width to control screenshot width. Resolved in priority order:
    1. Explicitly set in request options
    2. Auto-parsed from `<meta name="viewport" content="width=...">` in HTML
    3. Defaults to 800px if not specified and no meta tag found
  - viewport_height (int, optional): Custom viewport height to control screenshot height. Resolved in priority order:
    1. Explicitly set in request options
    2. Auto-parsed from `<meta name="viewport" content="height=...">` in HTML
    3. Defaults to 720px if not specified and no meta tag found
  - device_scale_factor_level (Literal["normal", "high", "ultra"], optional): Device pixel ratio level, default is "normal". Different levels use independent browser context pools for better performance and resource isolation.
    - `normal`: Device pixel ratio 1.0 (default)
    - `high`: Device pixel ratio 1.3
    - `ultra`: Device pixel ratio 1.8

### GET /text2img/data/{id}

Returns the corresponding image by id.
