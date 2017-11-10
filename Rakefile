desc 'build to destination folder'
task :build do
  sh 'bundle exec jekyll b'
end

desc 'add and commit'
task :commit do
  sh "git add . && git commit -m #{Time.now.strftime('Site updated at %F %T').inspect}"
end

def make_path_then_subl folder, file, ext, underscore = false
  file = '_'.concat file if underscore and file[0] != '_'
  file.concat ext if File.extname(file).empty?
  file = File.join folder, file
  touch file
  open(file, 'w') { |f| yield f } if block_given?
  sh "subl #{file}:999"
end

desc 'new sass file'
task :sass, [:file] do |t, args|
  make_path_then_subl '_sass', args.file, '.sass', true
end

desc 'new layout file'
task :layout, [:file] do |t, args|
  make_path_then_subl '_layouts', args.file, '.html'
end

desc 'new include file'
task :include, [:file] do |t, args|
  make_path_then_subl '_includes', args.file, '.html'
end

require 'stringex_lite'
desc 'new draft file'
task :draft, [:file] do |t, args|
  make_path_then_subl '_drafts', args.file.to_url, '.md' do |f|
    f.puts <<~EOF
      ---
      title: #{args.file}
      ---
    EOF
  end
end

def with_format file
  "#{Time.now.strftime('%Y-%m-%d')}-#{file.to_url}"
end
desc 'new post file'
task :post, [:file] do |t, args|
  make_path_then_subl '_posts', with_format(args.file), '.md' do |f|
    f.puts <<~EOF
      ---
      title: #{args.file}
      ---
    EOF
  end
end

require 'tty'
desc 'publish draft to post'
task :publish do |t, args|
  choices = Dir.glob('_drafts/*.*')
  if choices.empty?
    puts 'Not found any draft.'
    next
  end
  TTY::Prompt.new.multi_select('which draft(s) to publish?', choices).each do |draft|
    dest = draft.sub '_drafts/', "_posts/#{Time.now.strftime('%Y-%m-%d')}-"
    mv draft, dest
  end
end

desc 'unpublish post to draft'
task :unpublish do |t, args|
  choices = Dir.glob('_posts/*.*')
  if choices.empty?
    puts 'Not found any post.'
    next
  end
  TTY::Prompt.new.multi_select('which post(s) to unpublish?', choices).each do |post|
    dest = post.sub /_posts\/\d+\-\d+-\d+-/, '_drafts/'
    mv post, dest
  end
end