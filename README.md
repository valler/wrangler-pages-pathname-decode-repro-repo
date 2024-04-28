# Wrangler Pages Pathname Decode Repro Repo

## Observed behavior

Using `wrangler pages dev` for local development does not find files at paths
which require URL decoding and a redirect, e.g. when the pathname contains
`%C3%A4` (ä).

```sh
# ok
curl -L http://localhost:8788/index.html

# ok (client encodes as pathname as `/%C3%A4.txt`)
curl http://localhost:8788/ä.txt

# ok (client encodes as pathname as `/%C3%B6`)
curl http://localhost:8788/ö

# ok
curl http://localhost:8788/%C3%A4.txt

# ok
curl http://localhost:8788/%C3%A4/

# [wrangler:inf] GET /%e4/ 404 Not Found
curl -L  http://localhost:8788/%C3%A4/index.html

# [wrangler:inf] GET /%e4/ 404 Not Found
curl -L http://localhost:8788/%C3%A4

# [wrangler:inf] GET /%f6 404 Not Found
curl -L http://localhost:8788/ö.html

# [wrangler:inf] GET /%f6 404 Not Found
curl -L http://localhost:8788/ö/

# not found (expected)
curl http://localhost:8788/%E4.txt
```

## Expected behavior

It should find the files matching the decoded pathnames.

```sh
# ok
curl -L http://localhost:8788/index.html

# ok (client encodes as pathname as `/%C3%A4.txt`)
curl http://localhost:8788/ä.txt

# ok (client encodes as pathname as `/%C3%B6`)
curl http://localhost:8788/ö

# ok
curl http://localhost:8788/%C3%A4.txt

# ok
curl http://localhost:8788/%C3%A4/

# ok after redirect to `http://localhost:8788/%C3%A4/`
curl -L http://localhost:8788/%C3%A4/index.html

# ok after redirect to `http://localhost:8788/%C3%A4/`
curl -L http://localhost:8788/%C3%A4

# ok after redirect to `http://localhost:8788/%C3%B6`
curl -L http://localhost:8788/ö.html

# ok after redirect to `http://localhost:8788/%C3%B6`
curl -L http://localhost:8788/ö/

# 4xx (graceful handling of malformed URIs)
curl http://localhost:8788/%E4.txt
```

## Steps to reproduce

1. Clone the [repro repo](https://github.com/valler/wrangler-pages-pathname-decode-repro-repo).
2. Install dependencies: `npm install`
3. Start the server: `npx wrangler pages dev dist`.
4. In a second terminal run the commands as illustrated for the [observed behavior](#observed-behavior) or any alternative to fetch these URLs.
