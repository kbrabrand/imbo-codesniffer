require 'date'
require 'digest/md5'
require 'fileutils'
require 'nokogiri'

desc "Create a PEAR package"
task :generate_pear_package, :version do |t, args|
  version = args[:version]

  if /^[\d]+\.[\d]+\.[\d]+$/ =~ version
    now = DateTime.now
    hash = Digest::MD5.new
    xml = Nokogiri::XML::Builder.new do |xml|
      xml.package(
        :version => "2.0",
        :xmlns => "http://pear.php.net/dtd/package-2.0",
        "xmlns:tasks" => "http://pear.php.net/dtd/tasks-1.0",
        "xmlns:xsi" => "http://www.w3.org/2001/XMLSchema-instance",
        "xsi:schemaLocation" => ["http://pear.php.net/dtd/tasks-1.0", "http://pear.php.net/dtd/tasks-1.0.xsd", "http://pear.php.net/dtd/package-2.0", "http://pear.php.net/dtd/package-2.0.xsd"].join(" ")) {
        xml.name "ImboStandard"
        xml.channel "pear.starzinger.net"
        xml.summary "Imbo coding standard for PHP_CodeSniffer"
        xml.description "PHP_CodeSniffer coding standard used by Imbo and ImboClient"
        xml.lead {
          xml.name "Christer Edvartsen"
          xml.user "christeredvartsen"
          xml.email "cogo@starzinger.net"
          xml.active "yes"
        }
        xml.date now.strftime('%Y-%m-%d')
        xml.time now.strftime('%H:%M:%S')
        xml.version {
          xml.release version
          xml.api version
        }
        xml.stability {
          xml.release "stable"
          xml.api "stable"
        }
        xml.license "MIT", :uri => "http://www.opensource.org/licenses/mit-license.php"
        xml.notes "http://github.com/imbo/imbo-codesniffer/blob/#{version}/README.markdown"
        xml.contents {
          xml.dir(:name => "/", :baseinstalldir => "PHP/CodeSniffer/Standards") {
            `git ls-files *.php *.xml`.split("\n").each { |file|
              xml.file(:md5sum => hash.hexdigest(File.read(file)), :role => "php", :name => file)
            }
          }
        }
        xml.dependencies {
          xml.required {
            xml.php {
              xml.min "5.3.2"
            }
            xml.pearinstaller {
              xml.min "1.9.0"
            }
            xml.package {
              xml.name "PHP_CodeSniffer"
              xml.channel "pear.php.net"
              xml.min "1.3.0"
            }
          }
        }
        xml.phprelease
      }
    end

    # Write XML to package.xml
    File.open("package.xml", "w") { |f|
        f.write(xml.to_xml)
    }

    # Generate pear package
    system "pear package"

    File.unlink("package.xml")
  end
end

desc "Publish a PEAR package to pear.starzinger.net"
task :publish_pear_package, :version do |t, args|
  version = args[:version]

  if /^[\d]+\.[\d]+\.[\d]+$/ =~ version
    # Build PEAR package
    Rake::Task["generate_pear_package"].invoke(version)

    package = "ImboStandard-#{version}.tgz"

    if File.exists?(package)
      wd = Dir.getwd
      Dir.chdir("/home/christer/dev/christeredvartsen.github.com")
      system "git pull origin master"
      system "pirum add . #{wd}/#{package}"
      system "git add --all"
      system "git commit -am 'Added #{package[0..-5]}'"
      system "git push"
      Dir.chdir(wd)
    else
      puts "#{package} does not exist. Run the pear task first to create the package"
    end
  end
end

desc "Tag current state of the master branch and push it to GitHub"
task :tag_version, :version do |t, args|
  version = args[:version]

  if /^[\d]+\.[\d]+\.[\d]+$/ =~ version
    system "git checkout master"
    system "git tag #{version}"
    system "git push"
    system "git push --tags"
  end
end

desc "Release a new version (builds PEAR package, updates PEAR channel and pushes tag to GitHub)"
task :release, :version do |t, args|
  version = args[:version]

  if /^[\d]+\.[\d]+\.[\d]+$/ =~ version
    # Publish to the PEAR channel
    Rake::Task["publish_pear_package"].invoke(version)

    # Tag the current state of master and push to GitHub
    Rake::Task["tag_version"].invoke(version)
  end
end
