require 'html-proofer'

task :build do
  sh "JEKYLL_ENV=production bundle exec jekyll build --verbose"
end

task :test do
  options = {
    check_html: true,
    disable_external: true,
    empty_alt_ignore: true,
  }
  HTMLProofer.check_directory("./_site", options).run
end

task :deploy do
  sh "aws s3 sync _site/ s3://roberteshleman.com/ --delete"
  sh <<~EOF
    aws cloudfront create-invalidation \
      --distribution-id #{ENV['CLOUDFRONT_DISTRIBUTION_ID']} \
      --paths "/*"
  EOF
end

task :clean do
  rm_rf "_site"
end
