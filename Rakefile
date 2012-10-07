task :default => :server

desc "Clean up the _site folder"
task :cleanup do
  puts "Deleting _site folder ..."
  system "rm -rf _site/"
end

desc "Start jekyll server"
task :server => :cleanup do
  system 'jekyll --server --auto'
end

