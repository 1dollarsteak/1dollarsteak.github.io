source "https://rubygems.org"
git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem "github-pages"
gem "kramdown", ">= 2.3.0"

require 'rbconfig'
  if RbConfig::CONFIG['target_os'] =~ /(?i-mx:bsd|dragonfly)/
    gem 'rb-kqueue', '>= 0.2'
  end

group :jekyll_plugins do
  #gem 'jekyll-sitemap'
  #gem 'jekyll-feed'
  #gem 'jekyll-seo-tag'
  gem 'sassc'
end

