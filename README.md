# blog.kostic.dev

Repository for the blog where I share some stuff.

## Prerequisites

Follow the instructions in the [Jekyll Docs](https://jekyllrb.com/docs/installation/) to complete the installation of `Ruby`, `RubyGems`, `Jekyll` and `Bundler`.

### For Arch Linux specifically

```
sudo pacman -S ruby ruby-stdlib base-devel zlib 
gem install jekyll bundler
```

edit `~/.bashrc` or equivalent to add `export PATH="$HOME/.local/share/gem/ruby/3.3.0/bin:$PATH"`


Then run
```
source ~/.bashrc
bundle config set --local-path ~/.bundle
bundle install 
```

## Writing

When writing, it might come in handy to see how it would look. For this, we run

```zsh
bundle exec jekyll serve -l -o
```


## Usage

Please see the [theme's docs](https://github.com/cotes2020/jekyll-theme-chirpy#documentation).

## License

This work is published under [MIT][mit] License.

[gem]: https://rubygems.org/gems/jekyll-theme-chirpy
[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy/
[use-template]: https://github.com/cotes2020/chirpy-starter/generate
[CD]: https://en.wikipedia.org/wiki/Continuous_deployment
[mit]: https://github.com/cotes2020/chirpy-starter/blob/master/LICENSE
