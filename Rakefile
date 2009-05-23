require 'rake'
require 'rubygems'

ROOT = File.expand_path('.')
SRC = ROOT
DST = File.join(ROOT, 'build')
TMP = File.join(ROOT, 'tmp')

unless defined? OSX then
  OSX = PLATFORM =~ /darwin/
  WIN = PLATFORM =~ /win32/
  NIX = !(OSX || WIN)
end

begin
  require 'term/ansicolor'
  include Term::ANSIColor
rescue LoadError
  raise 'Run "gem install term-ansicolor"'
end
# http://kpumuk.info/ruby-on-rails/colorizing-console-ruby-script-output/
if WIN then
  begin
    require 'win32console'
    include Win32::Console::ANSI
  rescue LoadError
    raise 'Run "gem install win32console" to use terminal colors on Windows'
  end
end

#
# you can use FileUtils: http://corelib.rubyonrails.org/classes/FileUtils.html
#
require 'find'

def file_color(text); yellow(text); end
def dir_color(text); blue(text); end
def cmd_color(text); green(text); end

# copies directory tree without .svn, .git and other temporary files
def cp_dir(src, dst)
  puts "#{cmd_color('copying')} #{dir_color(src)}"
  puts "     -> #{dir_color(dst)}"
  Find.find(src) do |fn|
    next if fn =~ /\/\./
    r = fn[src.size..-1]
    if File.directory? fn
      mkdir File.join(dst,r) unless File.exist? File.join(dst,r)
    else
      cp(fn, File.join(dst,r))
    end
  end
end

def cp_file(src, dst)
  puts "#{cmd_color('copying')} #{file_color(src)}"
  puts "     -> #{file_color(dst)}"
  cp(src, dst)
end

def dep(src)
  s = File.expand_path src
  rs = s[SRC.size..-1]
  d = File.join(TMP, rs)
  puts "#{cmd_color('copying')} #{file_color(s)}"
  puts "     -> #{file_color(d)}"
  cp(s, d)
end

def my_mkdir(dir)
  puts "#{cmd_color('creating directory')} #{dir_color(dir)}"
  mkdir dir
end

def parse_version()
  f = File.new(File.join(SRC, 'install.rdf'))
  text = f.read
  unless text=~/<em:version>([^<]*)<\/em:version>/
    puts "#{red('Version not found')}"
    exit
  end
  $1
end

################################################################################

desc "prepare release XPI"
task :release do
  version = parse_version()
  $stderr = File.new('/dev/null', 'w') unless ENV["verbose"]

  remove_dir(TMP) if File.exists?(TMP) # recursive!
  mkdir(TMP)
  cp_dir(File.join(SRC, 'chrome'), File.join(TMP, "chrome"))
  cp_dir(File.join(SRC, 'defaults'), File.join(TMP, "defaults"))
  dep(File.join(SRC, 'chrome.manifest'))
  dep(File.join(SRC, 'install.rdf'))
  dep(File.join(SRC, 'license.txt'))
  my_mkdir(DST) unless File.exist?(DST)

  res = "#{DST}/firerainbow-#{version}.xpi"
  File.unlink(res) if File.exists?(res)
  puts "#{cmd_color('zipping')} #{file_color(res)}"
  Dir.chdir(TMP) do
    puts red('need zip on command line (download http://www.info-zip.org/Zip.html)') unless system("zip -r \"#{res}\" *");
  end
  remove_dir(TMP) if File.exist?(TMP) # recursive!
end

task :default => :release