require 'nanoc3/tasks'
require 'fileutils'

desc "Compile the site"
task :compile do
  `nanoc compile`
end

desc "Remove the output dir"
task :remove_output_dir do
  FileUtils.rm_r('output') if File.exist?('output')
end

def preview(browser=false)
  pids = [
    Process.spawn("bundle exec nanoc view"),
  ]

  if browser
    STDERR.puts "Opening a browser to preview the site..."
    STDERR.puts "  You may need to refresh the page if the server hasn't loaded yet."
    sleep 2
    pids << Process.spawn("bundle exec launchy http://localhost:3000")
  end

  Signal.trap "INT" do
    pids.each { |pid| Process.kill :INT, pid }
  end

  Process.waitall
end

desc "Build the site and host it localhost:3000"
task :preview do
  preview
end

namespace :preview do
  desc "Build the site and open localhost:3000 in the default browser"
  task :browser do
    preview true
  end
end

def commit_message
  publish_emojis = [':boom:', ':rocket:', ':metal:', ':bulb:', ':zap:',
    ':sailboat:', ':gift:', ':ship:', ':shipit:', ':sparkles:', ':rainbow:']
  default_message = "P U B L I S H #{publish_emojis.sample}"

  print "Enter a commit message (default: '#{default_message}'): "
  STDOUT.flush
  mesg = STDIN.gets.chomp.strip

  mesg = default_message if mesg == ''
  mesg.gsub(/'/, '') # Allow this to be handed off via -m '#{message}'
end

desc "Publish to http://www.icalhooks.com"
task :publish => [:clean, :remove_output_dir] do
  mesg = commit_message

  sh "nanoc compile"

  ENV['GIT_DIR'] = File.expand_path(`git rev-parse --git-dir`.chomp)
  old_sha = `git rev-parse refs/remotes/origin/gh-pages`.chomp
  Dir.chdir('output') do
    ENV['GIT_INDEX_FILE'] = gif = '/tmp/dev.gh.i'
    ENV['GIT_WORK_TREE'] = Dir.pwd
    File.unlink(gif) if File.file?(gif)
    `git add -A`
    tsha = `git write-tree`.strip
    puts "Created tree   #{tsha}"
    if old_sha.size == 40
      csha = `git commit-tree #{tsha} -p #{old_sha} -m '#{mesg}'`.strip
    else
      csha = `git commit-tree #{tsha} -m '#{mesg}'`.strip
    end
    puts "Created commit #{csha}"
    puts `git show #{csha} --stat`
    puts "Updating gh-pages from #{old_sha}"
    `git update-ref refs/heads/gh-pages #{csha}`
    `git push origin gh-pages`
  end
end
