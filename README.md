# [roberteshleman.com](https://roberteshleman.com)

[![CircleCI](https://circleci.com/gh/reshleman/roberteshleman.com/tree/master.svg?style=svg&circle-token=d323a779454b1f1d7a9df5775995e4e131bc891f)](https://circleci.com/gh/reshleman/roberteshleman.com/tree/master)

## Dependencies

Install with:

```
bundle install
```

## Development

To serve the site locally, run:

```
jekyll serve
```

By default, Jekyll serves the site at http://127.0.0.1:4000/.

## Build

To build the site for the "production" `JEKYLL_ENV`, run:

```
bundle exec rake build
```

## Testing

To run `html-proofer` against the generated site:

```
bundle exec rake test
```

This validates HTML and internal links.

## Deployment

[CircleCI](https://circleci.com/gh/reshleman/roberteshleman.com) builds and
deploys the site to S3 on a push or merge to `master`.

Deployment requires the `aws` CLI to be configured with an appropriate Access
Key ID and Secret Access Key, and the `CLOUDFRONT_DISTRIBUTION_ID` environment
variable to be set.

```
bundle exec rake deploy
```
