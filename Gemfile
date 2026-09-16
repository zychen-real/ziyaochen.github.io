source "https://rubygems.org"

# Jekyll 本体
gem "jekyll", "~> 4.3"

# 站点用到的插件
group :jekyll_plugins do
  gem "jekyll-paginate", "~> 1.1"   # 首页分页（index.html 里的 paginator）
  gem "jekyll-sitemap",  "~> 1.4"   # 自动生成 sitemap.xml
  gem "jekyll-feed",     "~> 0.17"  # 自动生成 feed.xml
  gem "jekyll-seo-tag",  "~> 2.8"   # SEO meta 标签
end

# Ruby 3.x 起从标准库剥离出来的 gem，缺了 Jekyll 会启动失败
gem "webrick", "~> 1.8"
gem "csv", "~> 3.3"
gem "base64", "~> 0.2"
gem "bigdecimal", "~> 3.1"
gem "logger", "~> 1.6"

# Windows / JRuby 下的时区数据
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end
