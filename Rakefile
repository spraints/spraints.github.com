require 'bundler/setup'


desc 'Build the site\'s css'
task :css => 'screen.css'

namespace :css do
  desc 'Auto-build the site\'s css to jekyll\'s output dir'
  task :watch do
    sh 'sass', '--scss', '--watch', 'sass/screen.scss:_site/screen.css'
  end
end

file 'screen.css' => Dir['sass/**/*.scss'] do
  sh 'sass', '--scss', 'sass/screen.scss', 'screen.css'
end
