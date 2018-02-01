# Copyright 2018 Google LLC
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

require "bundler/setup"
require "bundler/gem_tasks"

require "rubocop/rake_task"
RuboCop::RakeTask.new

desc "Run tests."
task :test do
  $LOAD_PATH.unshift "lib", "test"
  Dir.glob("test/**/*test.rb")
    .reject { |file| file.include? "smoke_test" }
    .each { |file| require_relative file }
end

namespace :test do
  desc "Runs tests with coverage."
  task :coverage do
    require "simplecov"
    SimpleCov.start do
      command_name "google-cloud-language"
      track_files "lib/**/*.rb"
      add_filter "test/"
    end

    Rake::Task["test"].invoke
  end
end

desc "Runs the smoke tests."
task :smoke_test do
  $LOAD_PATH.unshift "lib", "smoke_test"
  Dir.glob("acceptance/**/*smoke_test.rb").each { |file| require_relative file }
end

namespace :smoke_test do
  desc "Runs smoke tests with coverage."
  task :coverage do
    require "simplecov"
    SimpleCov.start do
      command_name "google-cloud-language"
      track_files "lib/**/*.rb"
      add_filter "test/"
    end

    Rake::Task["smoke_test"].invoke
  end
end

# Acceptance tests
desc "Run the google-cloud-language acceptance tests."
task :acceptance, :project, :keyfile do |t, args|
  project = args[:project]
  project ||=
    ENV["LANGUAGE_TEST_PROJECT"] ||
    ENV["GCLOUD_TEST_PROJECT"]
  keyfile = args[:keyfile]
  keyfile ||=
    ENV["LANGUAGE_TEST_KEYFILE"] ||
    ENV["GCLOUD_TEST_KEYFILE"]
  if keyfile
    keyfile = File.read keyfile
  else
    keyfile ||=
      ENV["LANGUAGE_TEST_KEYFILE_JSON"] ||
      ENV["GCLOUD_TEST_KEYFILE_JSON"]
  end
  if project.nil? || keyfile.nil?
    fail "You must provide a project and keyfile. e.g. rake acceptance[test123, /path/to/keyfile.json] or LANGUAGE_TEST_PROJECT=test123 LANGUAGE_TEST_KEYFILE=/path/to/keyfile.json rake acceptance"
  end
  require "google/cloud/language/credentials"
  (Google::Cloud::Language::Credentials::PATH_ENV_VARS +
   Google::Cloud::Language::Credentials::JSON_ENV_VARS).each do |path|
    ENV[path] = nil
  end
  ENV["LANGUAGE_PROJECT"] = project
  ENV["LANGUAGE_TEST_PROJECT"] = project
  ENV["LANGUAGE_KEYFILE_JSON"] = keyfile

  # Required for smoke tests
  ENV["SMOKE_TEST_PROJECT"] = project

  Rake::Task["acceptance:run"].invoke
end

namespace :acceptance do
  task :run do
    Rake::Task["smoke_test"].invoke
  end

  desc "Run acceptance tests with coverage."
  task :coverage do
  end

  desc "Run acceptance cleanup."
  task :cleanup do
  end
end

require "yard"
require "yard/rake/yardoc_task"
YARD::Rake::YardocTask.new

desc "Generates JSON output from google-cloud-language .yardoc"
task :jsondoc => :yard do
  require "rubygems"
  require "gcloud/jsondoc"

  registry = YARD::Registry.load! ".yardoc"

  toc_config = {
    documents: [
      {
        type: "toc",
        title: "Google::Cloud::Language::V1::DataTypes",
        modules: [
          {
            title: "Google::Cloud::Language::V1",
            include: ["google/cloud/language/v1"]
          },
          {
            title: "Google::Protobuf",
            include: ["google/protobuf"]
          },
          {
            title: "Google::Rpc",
            include: ["google/rpc"]
          }
        ]
      }
    ]
  }

  generator = Gcloud::Jsondoc::Generator.new registry,
                                             "google-cloud-language",
                                             generate: toc_config
  rm_rf "jsondoc", verbose: true
  generator.write_to "jsondoc"
  cp ["docs/toc.json"], "jsondoc", verbose: true
end

desc "Run yard-doctest example tests."
task :doctest do
  puts "The google-cloud-language gem does not have doctest tests."
end

desc "Run the CI build"
task :ci do
  header "BUILDING google-cloud-language"
  header "google-cloud-language rubocop", "*"
  sh "bundle exec rake rubocop"
  header "google-cloud-language jsondoc", "*"
  sh "bundle exec rake jsondoc"
  header "google-cloud-language doctest", "*"
  sh "bundle exec rake doctest"
  header "google-cloud-language test", "*"
  sh "bundle exec rake test"
end

namespace :ci do
  desc "Run the CI build, with acceptance tests."
  task :acceptance do
    Rake::Task["ci"].invoke
    header "google-cloud-language acceptance", "*"
    sh "bundle exec rake acceptance -v"
  end
  task :a do
    # This is a handy shortcut to save typing
    Rake::Task["ci:acceptance"].invoke
  end

  task :release, :tag do |t, args|
    tag = args[:tag]
    if tag.nil?
      fail "You must provide a tag to release."
    end
    # Verify the tag format "vVERSION"
    m = tag.match /\/v(?<version>\S*)/
    if m.nil? # We have a match!
      fail "Tag #{tag} does not match the expected format."
    end

    package = "google-cloud-language"
    version = m[:version]
    if version.nil?
      fail "You must provide a version."
    end

    api_token = ENV["RUBYGEMS_API_TOKEN"]

    require "gems"
    ::Gems.configure do |config|
      config.key = api_token
    end if api_token

    Bundler.with_clean_env do
      sh "rm -rf pkg"
      sh "bundle update"
      sh "bundle exec rake build"
    end

    path_to_be_pushed = "pkg/#{package}-#{version}.gem"
    if File.file? path_to_be_pushed
      begin
        ::Gems.push(File.new path_to_be_pushed)
        puts "Successfully built and pushed #{package} for version #{version}"

        Rake::Task["jsondoc:package"].invoke tag
      rescue => e
        puts "Error while releasing #{package} version #{version}: #{e.message}"
      end
    else
      fail "Cannot build #{package} for version #{version}"
    end
  end
end

namespace :travis do
  desc "Build for Travis-CI"
  task :build do
    run_acceptance = false
    if ENV["TRAVIS_BRANCH"] == "master" &&
        ENV["TRAVIS_PULL_REQUEST"] == "false"
      run_acceptance = true
    end

    Bundler.with_clean_env do
      sh "gem install bundler"
      sh "bundle update"

      if run_acceptance
        sh "bundle exec rake ci:acceptance"
      else
        sh "bundle exec rake ci"
      end
    end
  end
end

namespace :appveyor do
  desc "Build for AppVeyor"
  task :build do
    # Retrieve the SSL certificate from google-api-client gem
    ssl_cert_file = Gem.loaded_specs["google-api-client"].full_gem_path + "/lib/cacerts.pem"

    run_acceptance = false
    if ENV["APPVEYOR_REPO_BRANCH"] == "master" && !ENV["APPVEYOR_PULL_REQUEST_NUMBER"]
      run_acceptance = true
    end

    Bundler.with_clean_env do
      # Fix acceptance/data symlinks on windows
      require "fileutils"
      FileUtils.mkdir_p "acceptance"
      FileUtils.rm_f "acceptance/data"
      sh "call mklink /j acceptance\\data ..\\acceptance\\data"

      sh "bundle update"

      if run_acceptance
        # Set the SSL certificate so connections can be made
        ENV["SSL_CERT_FILE"] = ssl_cert_file

        sh "bundle exec rake ci:acceptance"
      else
        sh "bundle exec rake ci"
      end
    end
  end
end

task :default => :test

def header str, token = "#"
  line_length = str.length + 8
  puts ""
  puts token * line_length
  puts "#{token * 3} #{str} #{token * 3}"
  puts token * line_length
  puts ""
end
