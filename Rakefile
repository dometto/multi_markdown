require 'rake/clean'
require 'rdoc/task'
require 'bundler'
require 'fileutils'

Bundler::GemHelper.install_tasks

task :default => "test:unit"

# ***** Build

DLEXT = RbConfig::CONFIG['DLEXT']
MMD_DIR = File.expand_path("../ext/mmd", __FILE__)

# For Mac OS X -- prevents prevent additional ._* files being added to tarball
ENV['COPYFILE_DISABLE'] = 'true'

namespace "MultiMarkdown-6" do

  desc "Initialize the submodule"
  task "init" do
    FileUtils.rm_rf(MMD_DIR)
    chdir('MultiMarkdown-6') do
      sh 'make'

      ['Sources/libMultiMarkdown', 'build'].each do |dir|
        chdir(dir) do
          Dir.glob('{.,include}/*.{h,c}').each do |s|
            next if s =~ /template/
            dest = "#{MMD_DIR}/#{File.dirname(s)}"
            FileUtils.mkdir_p(dest)
            FileUtils.cp(s, dest)
          end
        end
      end

      token_h_file = "#{MMD_DIR}/include/token.h"
      IO.write(token_h_file, (File.open(token_h_file) do |f|
        f.read.gsub(/#define kUseObjectPool/, "#define kUseObjectPoolDisabled")
      end))

    end
  end

end

file 'ext/Makefile' => FileList['ext/{extconf.rb,*.c,*.h,*.rb}', "#{MMD_DIR}/*.{c,h}"] do
  chdir('ext') do
    ruby 'extconf.rb'
  end
end
CLEAN.include 'ext/Makefile'

file "ext/multi_markdown.#{DLEXT}" => FileList['ext/Makefile', 'ext/*.{c,h,rb}', "#{MMD_DIR}/*.{c,h}"] do |f|
  chdir('ext') do
    sh 'make'
  end
end
CLEAN.include 'ext/*.{o,bundle,so}'
CLEAN.include "#{MMD_DIR}/*.o"

file "lib/multi_markdown.#{DLEXT}" => "ext/multi_markdown.#{DLEXT}" do |f|
  cp f.prerequisites, "lib/", :preserve => true
end
CLEAN.include "lib/*.{so,bundle}"

desc 'Build the multi_markdown extension'
task :build => "lib/multi_markdown.#{DLEXT}"

# ***** Test

desc 'Run unit and conformance tests'
task :test => [ 'test:unit', 'test:conformance' ]

namespace :test do

  desc 'Run unit tests'
  task :unit => :build do |t|
    FileList['test/*_test.rb'].each do |f|
      ruby f
    end
  end

  desc "Run conformance tests"
  task :conformance => :build do |t|
    script = "#{pwd}/bin/rmultimarkdown"
    chdir("MultiMarkdown-6/MarkdownTest") do
      sh "./MarkdownTest.pl --script='#{script}' --flags='-c' --tidy"
      sh "./MarkdownTest.pl --script='#{script}' --testdir='MultiMarkdownTests'"
      sh "./MarkdownTest.pl --script='#{script}' --testdir='MultiMarkdownTests' --flags='-t latex' --ext='.tex'"
      sh "./MarkdownTest.pl --script='#{script}' --testdir='BeamerTests' --flags='-t latex' --ext='.tex'"
      sh "./MarkdownTest.pl --script='#{script}' --testdir='MemoirTests' --flags='-t latex' --ext='.tex'"
    end
  end

end

# ***** RDoc

Rake::RDocTask.new do |rd|
  rd.main = "README.md"
  rd.rdoc_files.include("README.md", "ext/**/*.c", "lib/**/*.rb")
end
