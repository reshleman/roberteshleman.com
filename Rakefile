task :deps do
  sh "bundle install"
end

task :build do
  sh "JEKYLL_ENV=production bundle exec jekyll build --verbose"
end

task :deploy do
  sh "aws s3 sync _site/ s3://roberteshleman.com/ --delete"
  # TODO: Invalidate Cloudfront cache
end

task :clean do
  rm_rf "_site"
end
