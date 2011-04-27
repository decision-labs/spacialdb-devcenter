require 'bundler/setup'

require 'rspec'
require 'rspec/core'
require 'rspec/core/rake_task'

desc "Run specs"
Rspec::Core::RakeTask.new(:spec) do |spec|
  spec.pattern    = "spec/**/*_spec.rb"
  spec.verbose    = true
  spec.rspec_opts = ['--color']
end