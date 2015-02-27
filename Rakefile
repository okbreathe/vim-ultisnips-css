desc "Make the sass/scss snippets"

task :build do
  require 'yaml'
  require 'fileutils'

  data  = YAML::load_file('snips.yml')
  snips = Snippets.new(data)

  [:css, :sass, :scss, :stylus].each do |format|
    fn = "UltiSnips/#{format}.snippets"

    File.open(fn, 'w') { |f| f.write snips.to_ultisnips(format) }
  end

  # Less is just an extension of CSS
  File.open("UltiSnips/less.snippets", 'w') { |f| f.write snips.extend_css  }

  [:scss, :css, :sass, :stylus].each do |format|
    snips.to_yasnippet(format).each do |snippet|
      dn = "Yasnippet/#{format}-mode/"
      fn = snippet[:name]
      FileUtils.mkdir_p(dn)
      File.open(dn + fn, 'w') { |f| f.write snippet[:snip] }
    end
  end
end

class Snippets
  attr_reader :snips

  def initialize(snips)
    @snips = snips
  end

  # Generate content for an Ultisnips file
  def to_ultisnips(format)
    lines = expand(format).map{|h| ultisnip_block(h) }
    lines.unshift(extend_css(-50)) if format == :scss
    lines.join("\n\n")
  end

  # Generate a list of yasnippet snippets
  def to_yasnippet(format)
    expand(format).map{|h| yasnippet_block(h) }
  end

  # Add priority header to certain file types
  def extend_css(priority = nil)
    ["extends css", ("priority #{priority}" if priority)].compact.join("\n")
  end

  # Expand our YAML into valid snippets
  def expand(format)
    out = []

    unless format == :scss
      out += @snips['simple'].map do |key, val|
        {
          name: key,
          desc: to_desc(val),
          snip: to_snippet(val, format)
        }
      end

      out += @snips['expressions'].map do |key, val|
        val = unplaceholder(val)
        {
          name: key,
          desc: to_desc(val),
          snip: val
        }
      end
    end

    if format == :sass || format == :scss
      out += @snips['sass-expressions'].map do |key, val|
        val = unplaceholder(val)
        {
          name: key,
          desc: to_desc(val),
          snip: val
        }
      end
    end

    if format == :stylus
      out += @snips['stylus-expressions'].map do |key, val|
        val = unplaceholder(val)
        {
          name: key,
          desc: to_desc(val),
          snip: val
        }
      end
    end

    unless format == :scss
      out += @snips['media'].map do |key, val|
        {
          name: key,
          desc: val,
          snip: (brackety?(format) ? val.gsub(') ', ') { ') : val)
        }
      end
    end

    out += @snips['css3'].map do |key, val|
      {
        name: key,
        desc: to_desc(val),
        snip: to_snippet(val, format, true)
      }
    end

    out
  end

  private

  def brackety?(format)
    format == :css || format == :scss
  end

  def to_desc(snippet)
    snippet.gsub(/\{(.*?)\}/, '___')
  end

  def unplaceholder(snippet)
    i = 0
    snippet.gsub(/\{(.*?)\}/) { |placeholder| "${#{i += 1}:#{placeholder[1...-1]}}" }
  end

  # Turns a raw snippet into a snippet of a given format
  def to_snippet(value, format, mixinify=false)
    snippet = unplaceholder(value.dup.gsub(/; /, "\n"))

    if mixinify
      if format == :sass
        snippet.gsub!(/^(.*?): (.*?)$/, "+\\1(\\2)")
      elsif format == :scss
        snippet.gsub!(/^(.*?): (.*?)$/, "@include \\1(\\2)")
      end
    end

    snippet.gsub!(/$/, ';')  if brackety?(format)

    snippet
  end

  # Formats a ultisnip block
  def ultisnip_block(options)
    [
      "snippet #{options[:name]} \"#{options[:desc]}\"",
      options[:snip],
      "endsnippet"
    ].join("\n")
  end

  # Formats a yasnippet block
  def yasnippet_block(options)
    options.merge(snip: [ 
      "# -*- mode: snippet -*-",
      "#name : #{options[:desc]}", 
      "# --", 
      options[:snip] 
    ].join("\n"))
  end
end
