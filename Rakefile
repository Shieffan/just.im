require 'digest'

task :default => :generate

desc 'Create new post with rake "post[post-name]"'
task :post, [:title] do |t, args|
  if args.title then
    new_post(args.title,'article')
  else
    puts 'rake "post[post-name]"'
  end
end

desc 'Create new reading with rake "reading[reading-name]"'
task :reading, [:title] do |t, args|
  if args.title then
    new_post(args.title,'reading')
  else
    puts 'rake "reading[reading-name]"'
  end
end

desc 'Build site with Jekyll'
task :generate => [:clean] do
  `jekyll`
end

desc 'Start server'
task :server => [:clean] do
  `jekyll serve -t`
end

desc 'Deploy with rake "depoly[comment]"'
task :deploy, [:comment] => :generate do |t, args|
  update_static_version
  if args.comment then
    `git commit . -m '#{args.comment}' && git push`
  else
    `git commit . -m 'new deployment' && git push`
  end
end

desc 'Clean up'
task :clean do
  `rm -rf _site`
end

def new_post(title,type)
  time = Time.now
  filename = type+"/_posts/" + time.strftime("%Y-%m-%d-") + title + '.markdown'
  if File.exists? filename then
    puts "Post already exists: #{filename}"
    return
  end
  uuid = `uuidgen | tr "[:upper:]" "[:lower:]" | tr -d "\n"`
  File.open(filename, "wb") do |f|
    f << <<-EOS
---
title: #{title}
layout: post
guid: urn:uuid:#{uuid}
tags:
  - 
---


EOS
  end
  puts "created #{filename}"
  `git add #{filename}`
end



def update_static_version
  md5 = Digest::MD5.hexdigest(File.read('static/css/compressed.css'))
  return true if File.file?('static/css/compressed-'+md5+'.css');
  puts 'compressed.css has been modified, updating static version..'
  `rm -f static/css/compressed-*.css`
  `cp -f static/css/compressed.css  static/css/compressed-#{md5}.css`
  `sed -i -e 's/compressed.*\.css/compressed-#{md5}.css/' _layouts/default.html`
  `git add static/css/compressed-#{md5}.css`
  puts 'Update static file version successfully'
end

