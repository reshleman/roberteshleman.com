# [roberteshleman.com](https://roberteshleman.com)

[![CircleCI](https://circleci.com/gh/reshleman/roberteshleman.com/tree/master.svg?style=svg&circle-token=d323a779454b1f1d7a9df5775995e4e131bc891f)](https://circleci.com/gh/reshleman/roberteshleman.com/tree/master)

A simple React resume/CV site built with Vite.

## Dependencies

Install with:

```
npm install
```

## Development

To serve the site locally, run:

```
npm run dev
```

Vite serves the site at http://localhost:5173/.

## Build

To build the site for production:

```
npm run build
```

This outputs the built site to `dist/`.

## Deployment

[CircleCI](https://circleci.com/gh/reshleman/roberteshleman.com) builds and
deploys the site to S3 on a push or merge to `master`.

Deployment requires the following environment variables to be set in CircleCI:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `CLOUDFRONT_DISTRIBUTION_ID`

To deploy manually:

```
npm run deploy
```
