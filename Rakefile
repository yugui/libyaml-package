TMP_DIR = File.expand_path('./destroot')
EDITABLE_FILES = %w[ lib/libyaml.la lib/sparcv9/libyaml.la ]
TARGET_VERSION = '0.1.3'
TARBALL = "yaml-#{TARGET_VERSION}.tar.gz"
autoload :ERB, 'erb'

task :default => :package

task :package => %w[ prototype postinstall make_relocatable ] do
  pwd = Dir.pwd
  sh "pkgmk -o -r #{TMP_DIR} -a `uname -p` -d #{pwd}/pkg"
  sh "pkgtrans -s #{pwd}/pkg #{pwd}/pkg/YUGUIlibyaml.pkg YUGUIlibyaml"
end

task :make_relocatable do
  EDITABLE_FILES.each do |path|
    path = File.join(TMP_DIR, path)
    contents = File.read(path).gsub(TMP_DIR, '##PREFIX##')
    File.open(path, "w"){|f| f.write contents }
  end
end

task :postinstall do
  erb = ERB.new(File.read("postinstall.erb"))
  File.open("postinstall", "w"){|f| f.write erb.result }
end

task :prototype => :build do
  File.open("prototype", "w"){|f|
    f.write <<-EOS.gsub(/^\s+/, '')
      i pkginfo
      i LICENSE=yaml-#{TARGET_VERSION}/LICENSE
      i postinstall=./postinstall
    EOS

    IO.popen("find #{TMP_DIR} | pkgproto", "r"){|pipe|
      pipe.each do |line|
        next if /d none #{TMP_DIR} [0-7]{4} \w+ \w+/ =~ line
        line = line.sub("#{TMP_DIR}/", '')
        line = line.sub(/([0-7]{4}) \w+ \w+/, '\1 root root')

        if EDITABLE_FILES.any?{|path| line == "f none #{path} 0755 root root\n" }
          f.write line.sub(/^f/, 'e')
        else
          f.write line
        end
      end
    }
  }
end

namespace :build do
  [['32', 'lib'], ['64', 'lib/sparcv9']].each do |arch, libdir|
    task arch => TARBALL do
      dir = "yaml-#{TARGET_VERSION}"

      rm_rf dir
      sh "gzip -dc #{TARBALL} | tar xf -"
      Dir.chdir(dir){
        sh "./configure --prefix=#{TMP_DIR} MAKE=gmake CC=cc CFLAGS='-g -m#{arch}' --libdir=#{TMP_DIR}/#{libdir}"
        sh "gmake"
        sh "gmake install"
      }
    end
  end
end

file TARBALL do
  sh "curl 'http://pyyaml.org/download/libyaml/#{TARBALL}' > #{TARBALL}"
end

task :build do
  rm_rf TMP_DIR
  Rake::Task['build:32'].invoke
  Rake::Task['build:64'].invoke
end
