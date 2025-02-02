require "rubygems"
require "yaml"
require "html-proofer"

desc "Build website"
task :build do
  puts "## Running: bundle exec jekyll build --verbose --trace"
  system "bundle exec jekyll build --verbose --trace"
  cd "_site" do
    puts "## Showing changes in _site"
    system "git status"
    puts "## Build Complete!"
  end
end

desc "Run local server"
namespace :serve do

  desc "Run local server with _config-dev.yml"
  task :dev do
    begin
      puts "## Running: bundle exec jekyll serve --config _config-dev.yml --host 0.0.0.0"
      config_prod=YAML.load_file("_config.yml")
      config_prod.merge!({ "baseurl" => nil })
      config_prod.each do |key, value|
        if key == "defaults"
          value[2].each do |key, value| # collections.radios.published = false
            key == "values" ? value.merge!({ "published" => false }) : nil
          end
          value[3].each do |key, value| # collections.cctv.published = false
            key == "values" ? value.merge!({ "published" => false }) : nil
          end
        end
      end
      config_dev=File.new("_config-dev.yml","w")
      config_dev.write(config_prod.to_h.to_yaml)
      config_dev.close
      system "bundle exec jekyll serve --config _config-dev.yml --host 0.0.0.0"
    rescue Exception => e
      puts e
      puts "\n## Shutting down server"
    end
  end

  desc "Run local server with _config.yml"
  task :prod do
    begin
      puts "## Running: bundle exec jekyll serve --host 0.0.0.0"
      system "bundle exec jekyll serve --host 0.0.0.0"
    rescue Exception => e
      puts e
      puts "\n## Shutting down server"
    end
  end

  desc "Run local server without a collection"
  namespace :without do

    desc "Run local server without radios collection"
    task :radios do
      begin
        puts "## Running: bundle exec jekyll serve --config _config-dev.yml --host 0.0.0.0"
        config_prod=YAML.load_file("_config.yml")
        config_prod.each do |key, value|
          if key == "defaults"
            value[2].each do |key, value| # collections.radios.published = false
              key == "values" ? value.merge!({ "published" => false }) : nil
            end
          end
        end
        config_dev=File.new("_config-dev.yml","w")
        config_dev.write(config_prod.to_h.to_yaml)
        config_dev.close
        system "bundle exec jekyll serve --config _config-dev.yml --host 0.0.0.0"
      rescue Exception => e
        puts e
        puts "\n## Shutting down server"
      end
    end

    desc "Run local server without cctv collection"
    task :cctv do
      begin
        puts "## Running: bundle exec jekyll serve --config _config-dev.yml --host 0.0.0.0"
        config_prod=YAML.load_file("_config.yml")
        config_prod.each do |key, value|
          if key == "defaults"
            value[3].each do |key, value| # collections.cctv.published = false
              key == "values" ? value.merge!({ "published" => false }) : nil
            end
          end
        end
        config_dev=File.new("_config-dev.yml","w")
        config_dev.write(config_prod.to_h.to_yaml)
        config_dev.close
        system "bundle exec jekyll serve --config _config-dev.yml --host 0.0.0.0"
      rescue Exception => e
        puts e
        puts "\n## Shutting down server"
      end
    end

  end
end

desc "Deploy tasks"
namespace :deploy do

  desc "Deploy site to Github Pages"
  task :website do
    puts "## Deploying to GitHub Pages"

    puts "## Generating site"
    system "bundle exec jekyll build"

    cd "_site" do
      system "git add -A"

      message = "Site updated at #{Time.now.utc}"
      puts "## Commiting: #{message}"
      system "git commit -m \"#{message}\""

      puts "## Pushing generated site"
      system "git push"

      puts "## Deploy Complete!"
    end
  end

  desc "Deploy to Rubygems a new gem version"
  task :"new-version" do
    ARGV.each { |a| task a.to_sym do ; end }

    puts "## Running: bump #{ARGV[1]} --tag"
    system "bump #{ARGV[1]} --tag"
    new_version = `bump current | sed -e 's/^.*:.//g' | tr -d '\n'`

    puts "## Running: git push origin master"
    system "git push origin master"

    puts "## Running: git push origin v#{new_version}" # Push tagged version
    system "git push origin v#{new_version}"

    puts "## Running: gem build #{File.basename(Dir.pwd)}.gemspec"
    system "gem build #{File.basename(Dir.pwd)}.gemspec"

    puts "## Running: gem push #{File.basename(Dir.pwd)}-#{new_version}.gem"
    system "gem push #{File.basename(Dir.pwd)}-#{new_version}.gem"
  end

end

desc "Git reset commands"
namespace :reset do
  desc "Git reset soft"
  task :soft do
    puts "## Current last commit: " + `git log --oneline | head -n 1`

    puts "## Rolling back last commit (git reset --soft HEAD~1)"
    system "git reset --soft HEAD~1"

    puts "## Current last commit: " + `git log --oneline | head -n 1`
  end
end

def edit_contents_file(path, text_to_search, old_string, new_string)
  temp_file = Tempfile.new('foo')
  begin
    File.open(path, 'r') do |file|
      file.each_line do |line|
        temp_file.puts line.include?(text_to_search) ? line.gsub(old_string, new_string) : line
      end
    end
    temp_file.close
    FileUtils.mv(temp_file.path, path)
  ensure
    temp_file.close
    temp_file.unlink
  end
end

def htmlproofer_ignore_canonical(old_string, new_string)
  gem_path = `bundle show jekyll-seo-tag`.strip
  template_path = "#{gem_path}/lib/template.html"
  template_line = "rel=\"canonical\""
  edit_contents_file(template_path, template_line, old_string, new_string)
end

desc "Continuous integration tests"
namespace :test do
  desc "Test HTML with html-proofer"
  task :html do
    puts "## Skipping rel=\"canonical\" from test"
    htmlproofer_ignore_canonical /\/>/, "data-proofer-ignore \/>"

    puts "## Running: bundle exec jekyll build --trace"
    sh "bundle exec jekyll build --trace"

    puts "## Revert rel=\"canonical\" node to stock config"
    htmlproofer_ignore_canonical /data-proofer-ignore \/>/, "\/>"

    def get_options()
      default_options = {
        :assume_extension => true,
        :error_sort => :desc,
        :url_ignore => ["https://mx.linkedin.com/company/grupo-pv-mexico"],
       }

      tasks_options = {
        :internal => { :disable_external => true },
        :external => { :external_only => true },
      }

      ARGV.each { |a| task a.to_sym do ; end }
      tasks_options.each do |key, value|
        if key.to_s == ARGV[1]
          default_options.merge!(value)
        end
      end

      return default_options
    end

    HTMLProofer.check_directory("./_site", get_options).run
  end
end

def alias_task(tasks)
    tasks.each do |new_name, old_name|
        task new_name, [*Rake.application[old_name].arg_names] => [old_name]
    end
end

alias_task [
    [:b, :build],
    [:sd, 'serve:dev'],
    [:sp, 'serve:prod'],
    [:swr, 'serve:without:radios'],
    [:swc, 'serve:without:cctv'],
    [:dw,  'deploy:website'],
    [:dnv, 'deploy:new-version'],
    [:rs, 'reset:soft'],
    [:th, 'test:html'],

    [:deploy, 'deploy:new-version'],
    [:full, 'serve:prod'],
    [:test, 'test:html'],
]

desc "Run local server with _config-dev.yml"
task :default => [:sd]
