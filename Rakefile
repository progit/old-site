require 'rubygems'
require 'erb'
require 'maruku'
require 'pp'

def update_toc(lang, toc)
  # update all the [[nav-prev]], [[nav-next]] links
  # write a table of contents

  pages = ['index.html']
  index = "<ul id=\"toc\">\n"
  toc.each do |chapter|
    (ch, data) = chapter
    next if data[:title].size == 0
    index += "<li><h1>#{ch}. <a href=\"ch#{ch}-0.html\">#{data[:title]}</a></h1></li>\n"
    index += "<ul>"
    pages << "ch#{ch}-0.html"
    data[:sections].each do |section|
      next if section[0] == 0
      index += "<li>#{ch}.#{section[0]} - <a href=\"ch#{ch}-#{section[0]}.html\">#{section[1]}</a></li>"
      pages << "ch#{ch}-#{section[0]}.html"
    end
    index += "</ul>"
  end
  index += "<li><h1><a href=\"commands.html\">Index of Commands</a></h1></li>"
  index += "</ul>"
  pages << "index.html"

  layout = ['en', 'ko'].include?(lang) ? 'master' : 'translation'

  html = "---
layout: #{layout}
title: Table of Contents
---
"

  html += '
<div id="buy-book-toc">
  <center>
    <a href="http://www.amazon.com/gp/product/1430218339?ie=UTF8&tag=prgi-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=1430218339"><img border="0" src="/images/book-big.jpg"></a><img src="http://www.assoc-amazon.com/e/ir?t=prgi-20&l=as2&o=1&a=1430218339" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
  </center>
  <p>Support this site by buying a print version of
  <a href="http://www.amazon.com/gp/product/1430218339?ie=UTF8&tag=prgi-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=1430218339">Pro Git</a><img src="http://www.assoc-amazon.com/e/ir?t=prgi-20&l=as2&o=1&a=1430218339" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
  <p><a href="http://twitter.com/chacon"><img src="/images/twitterbird.png"></a><a href="http://twitter.com/chacon">Follow the author</a> on Twitter for updates and Git tips</p>'
  if lang == 'ko'
    html += '
  <p>
  <h3>Also available in:</h3>
    <ul>
      <li><a href="http://dogfeet.github.com/progit/progit.ko.pdf">Korean PDF</a></li>
      <li><a href="http://dogfeet.github.com/progit/progit.ko.mobi">Korean Mobi</a></li>
      <li><a href="http://dogfeet.github.com/progit/progit.ko.epub">Korean Epub</a></li>
    </ul>
  </p>'
  else
    html += '
  <p>
  <h3>Also available in:</h3>
    <ul>
      <li><a href="/ebook/progit.pdf">PDF</a></li>
      <li><a href="https://github.s3.amazonaws.com/media/pro-git.en.mobi">Mobi</a></li>
      <li><a href="https://github.s3.amazonaws.com/media/progit.epub">Epub</a></li>
    </ul>
  </p>'
  end
  html += '
</div>
'
  html += index

  if lang == 'en'
    File.open('../book/index.html', 'w+') { |f| f.write(html) }
  else
    print `pwd`
    File.open("../book/#{lang}/index.html", 'w+') { |f| f.write(html) }
  end

  aindex = {}

  i = 1
  while i < (pages.size - 1)
    filter = pages[i]
    prevp = pages[i - 1]
    nextp = pages[i + 1]
    if lang == 'en'
      content = File.read("../book/#{filter}")
    else
      content = File.read("../book/#{lang}/#{filter}")
    end
    content.gsub!('[[nav-prev]]', prevp)
    content.gsub!('[[nav-next]]', nextp)

    content.scan(/git (\-+[a-z\-=]+ )*([a-z][a-z\-]+)/) do |match|
      aindex[match[1]] ||= {}
      aindex[match[1]][filter] ||= {:count => 0}
      aindex[match[1]][filter][:count] += 1
    end

    if lang == 'en'
      File.open("../book/#{filter}", 'w+') { |f| f.write(content) }
    else
      File.open("../book/#{lang}/#{filter}", 'w+') { |f| f.write(content) }
    end
    i += 1
  end

  ignore = ['aware', 'binaries', 'ci', 'co', 'com', 'directory', 'feature',
           'gitolite', 'gitosis-init', 'installed', 'last', 'library', 'my',
           'mygrit', 'project', 'prune', 'rack', 'repository', 'stash-unapply',
           'tarball', 'that', 'user', 'visual', 'will', 'world', 'unstage']

  groups = [
       ['Setup and Config', [ 'config', 'help' ]],
       ['Getting and Creating Projects', [ 'init', 'clone']],
       ['Basic Snapshotting', [ 'add', 'status', 'diff', 'commit', 'reset', 'rm', 'mv']],
       ['Branching and Merging', [ 'branch', 'checkout', 'merge', 'mergetool', 'log', 'stash', 'tag' ]],
       ['Sharing and Updating Projects', [ 'fetch', 'pull', 'push', 'remote', 'submodule' ]],
       ['Inspection and Comparison', [ 'show', 'log', 'diff', 'shortlog', 'describe' ]],
       ['Patching', ['am', 'apply', 'cherry-pick', 'rebase']],
       ['Debugging', [ 'bisect', 'blame' ]],
       ['Email', ['am', 'apply', 'format-patch', 'send-email', 'request-pull']],
       ['External Sytems', ['svn', 'fast-import']],
       ['Administration', [ 'gc', 'fsck', 'reflog', 'filter-branch', 'instaweb', 'archive' ]],
       ['Server Admin', [ 'daemon', 'update-server-info' ]],
      ]
  
  commands = "---
layout: #{layout}
title: Index of Commands
---
<div class=\"commands-index\">
"
  com = aindex.keys.sort

  groups.each do |group|
    pp group
    desc, comms = group
    commands << "<h2>#{desc}</h2>"
    commands << "<ul>"
    comms.each do |c|
      commands << "<li><strong>#{c}</strong> &nbsp; <small>#{links(aindex[c])}</small></li>"
      com.delete(c)
    end
    commands << "</ul>"
  end

  commands << "<h2>Plumbing Commands</h2>"
  commands << "<ul>"
  com.each do |command|
    next if ignore.include?(command)
    commands << "<li><strong>#{command}</strong> &nbsp; <small>#{links(aindex[command])}</small></li>"
  end
  commands << "</ul>"

  commands << "</div>"

  if lang == 'en'
      File.open("../book/commands.html", 'w+') {|f| f.write(commands)}
  else
      File.open("../book/#{lang}/commands.html", 'w+') {|f| f.write(commands)}
  end

end

def links(pages)
  return '' if !pages
  pages.sort.map do |page, data|
    d = page.scan(/ch(.*?).html/).first.first
    "<span class=\"pagelink\"><a href=\"#{page}\">#{d}</a> (#{data[:count]})</span>\n"
  end.join(' ')
end

def generate_pages(lang, chapter, content)
  #pname = "p/#{page}.html"
  #out = ERB.new(File.read('template/page.erb.html')).result

  toc = {:title => '', :sections => []}

  doc = Maruku.new(content)

  raw = doc.to_html
  if m = raw.match(/<h1(.*?)>(.*?)<\/h1>/)
    chapter_title = m[2]
    toc[:title] = chapter_title
  end


  # replace images
  if images = raw.scan(/Insert (.*?).png/)
    images.each do |img|
      real = "<center><img src=\"/figures/ch#{chapter}/#{img}-tn.png\"></center><br/>"
      raw.gsub!("Insert #{img}.png", real)
    end
  end

  sections = raw.split('<h2')
  section = 0
  sections.each do |sec|

    section_title = ''
    if section_match = sec.match(/>(.*?)<\/h2>/)
      section_title = section_match[1]
      toc[:sections] << [section, section_title]
    else
      toc[:sections] << [section, nil]
    end

    if lang == 'en'
      pname = "../../../book/ch#{chapter}-#{section}.html"
    else
      FileUtils.mkdir("../../../book/#{lang}") rescue nil
      pname = "../../../book/#{lang}/ch#{chapter}-#{section}.html"
    end

    full_title = section_match ? "#{chapter_title} #{section_title}" : chapter_title
    layout = ['en', 'ko'].include?(lang) ? 'master' : 'translation'
    html = "---
layout: #{layout}
title: Pro Git #{chapter}.#{section} #{full_title}
---
"
    if section_match
      sec = '<h2' + sec
    else
      html += "<h1>Chapter #{chapter}</h1>"
    end

    html += sec

    nav = "<div id='nav'>
<a href='[[nav-prev]]'>prev</a> | <a href='[[nav-next]]'>next</a>
</div>"
    html += nav

    File.open(pname, 'w+') { |f| f.write(html) }
    section += 1
  end
  toc
end

# generate the site
desc "Generate the book html for the site"
task :genbook do

  genlang = ENV['GENLANG']

  # git read-tree --prefix=book-content/ -u gitbook/master
  # git rm -rf book-content/

  Dir.chdir('translations') do
    Dir.glob("*").each do |lang|
      chapter_number = 0
      toc = []
      next if genlang && genlang != lang
      Dir.chdir(lang) do
        Dir.glob("*").each do |chapter|
          puts 'generating : ' + lang + '/' + chapter
          content = ''
          Dir.chdir(chapter) do
            Dir.glob('*').each do |section|
              content += File.read(section)
            end
            chapter_number += 1
            toc << [chapter_number, generate_pages(lang, chapter_number, content)]
          end rescue nil
        end
      end
      update_toc(lang, toc)
    end
  end

end

# generate the site
desc "Convert images"
task :convert_images do
  Dir.chdir('figures') do
    Dir.glob("*").each do |chapter|
      Dir.chdir(chapter) do
        Dir.glob("*").each do |image|
          puts image
          (im, ending) = image.split('.')
          if ending == 'png' and im[-3, 3] != '-tn'
            convert_image = "#{im}-tn.png"
            if !File.exists?(convert_image)
              width_out = `exiftool #{image} | grep 'Image Width'`
              width = width_out.scan(/: (\d+)/).first.first.to_i
              if width > 500
                `convert -thumbnail 500x #{image} #{convert_image}`
              else
                `cp #{image} #{convert_image}`
              end
            end
          end
        end
      end
    end
  end
end

task :default => [:genbook]
