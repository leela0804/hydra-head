require 'rake/testtask'
require 'bundler'
Bundler::GemHelper.install_tasks

APP_ROOT= File.dirname(__FILE__)
require 'jettywrapper'

namespace :jetty do
  TEMPLATE_DIR = 'hydra-core/lib/generators/hydra/templates'
  SOLR_DIR = "#{TEMPLATE_DIR}/solr_conf/conf"
  FEDORA_DIR = "#{TEMPLATE_DIR}/fedora_conf/conf"

  desc "Config Jetty"
  task :config do
    Rake::Task["jetty:reset"].reenable
    Rake::Task["jetty:reset"].invoke
    Rake::Task["jetty:config_fedora"].reenable
    Rake::Task["jetty:config_fedora"].invoke
    Rake::Task["jetty:config_solr"].reenable
    Rake::Task["jetty:config_solr"].invoke
  end
  
  desc "Copies the default SOLR config for the bundled Hydra Testing Server"
  task :config_solr do
    FileList["#{SOLR_DIR}/*"].each do |f|  
      cp("#{f}", 'jetty/solr/development-core/conf/', :verbose => true)
      cp("#{f}", 'jetty/solr/test-core/conf/', :verbose => true)
    end

  end

  desc "Copies a custom fedora config for the bundled Hydra Testing Server"
  task :config_fedora do
    # load a custom fedora.fcfg - 
    if defined?(Rails.root)
      app_root = Rails.root
    else
      app_root = File.join(File.dirname(__FILE__),"..")
    end
     
    fcfg = File.join(FEDORA_DIR,"development","fedora.fcfg")
    if File.exists?(fcfg)
      puts "copying over development/fedora.fcfg"
      cp("#{fcfg}", 'jetty/fedora/default/server/config/', :verbose => true)
    else
      puts "#{fcfg} file not found -- skipping fedora config"
    end
    fcfg = File.join(FEDORA_DIR,"test","fedora.fcfg")
    if File.exists?(fcfg)
      puts "copying over test/fedora.fcfg"
      cp("#{fcfg}", 'jetty/fedora/test/server/config/', :verbose => true)
    else
      puts "#{fcfg} file not found -- skipping fedora config"
    end
  end

  desc "Copies the default SOLR config files and starts up the fedora instance."
  task :load => [:config, 'jetty:start']

  desc "return development jetty to its pristine state, as pulled from git"
  task :reset => ['jetty:stop'] do
    system("cd jetty && git reset --hard HEAD && git clean -dfx && cd ..")
    sleep 2
  end
end

desc "Run Continuous Integration"
task :ci => ['jetty:reset', 'jetty:config'] do
  jetty_params = Jettywrapper.load_config
  error = nil
  error = Jettywrapper.wrap(jetty_params) do
    Rake::Task['spec'].invoke
  end
  raise "test failures: #{error}" if error

  Rake::Task["doc"].invoke
  

end

task :spec do
  raise "test failures" unless all_modules('rake spec')
end
task :clean do
  raise "test failures" unless all_modules('rake clean')
end

def all_modules(cmd)
  ['hydra-access-controls', 'hydra-core', 'hydra-file-access'].each do |dir|
    Dir.chdir(dir) do
      puts "\n\e[1;33m[Hydra CI] #{dir}\e[m\n"
      #cmd = "bundle exec rake spec" # doesn't work because it doesn't require the gems specified in the Gemfiles of the test rails apps 
      return false unless system(cmd)
    end
  end
end

begin
  require 'yard'
  require 'yard/rake/yardoc_task'
  project_root = File.expand_path(".")
  doc_destination = File.join(project_root, 'doc')
  if !File.exists?(doc_destination) 
    FileUtils.mkdir_p(doc_destination)
  end

  YARD::Rake::YardocTask.new(:doc) do |yt|
    yt.files   = ['*/lib/**/*.rb', project_root+"*", '*/app/**/*.rb']

    yt.options << "-m" << "textile"
    yt.options << "--protected"
    yt.options << "--no-private"
    yt.options << "-r" << "README.textile"
    yt.options << "-o" << "doc"
    yt.options << "--files" << "*.textile"
  end
rescue LoadError
  desc "Generate YARD Documentation"
  task :doc do
    abort "Please install the YARD gem to generate rdoc."
  end
end
