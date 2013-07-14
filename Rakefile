# -*- ruby -*-

require 'rubygems'
require 'hoe'

Hoe.plugin :seattlerb
Hoe.plugin :gemspec

Hoe.spec 'minitest' do
  developer 'Ryan Davis', 'ryand-ruby@zenspider.com'

  self.rubyforge_name = "bfts"
  self.testlib = :minitest
end

desc "Find missing expectations"
task :specs do
  $:.unshift "lib"
  require "minitest/unit"
  require "minitest/spec"

  pos_prefix, neg_prefix = "must", "wont"
  skip_re = /^(must|wont)$|wont_(throw)|must_(block|not?_|nothing|raise$)/x
  dont_flip_re = /(must|wont)_(include|respond_to)/

  map = {
    /(must_throw)s/                        => '\1',
    /(?!not)_same/                         => '_be_same_as',
    /_in_/                                 => '_be_within_',
    /_operator/                            => '_be',
    /_includes/                            => '_include',
    /(must|wont)_(.*_of|nil|silent|empty)/ => '\1_be_\2',
    /must_raises/                          => 'must_raise',
  }

  expectations = Minitest::Expectations.public_instance_methods.map(&:to_s)
  assertions   = Minitest::Assertions.public_instance_methods.map(&:to_s)

  assertions.sort.each do |assertion|
    expectation = case assertion
                  when /^assert/ then
                    assertion.sub(/^assert/, pos_prefix.to_s)
                  when /^refute/ then
                    assertion.sub(/^refute/, neg_prefix.to_s)
                  end

    next unless expectation
    next if expectation =~ skip_re

    regexp, replacement = map.find { |re, _| expectation =~ re }
    expectation.sub! regexp, replacement if replacement

    next if expectations.include? expectation

    args = [assertion, expectation].map(&:to_sym).map(&:inspect)
    args << :reverse if expectation =~ dont_flip_re

    puts
    puts "##"
    puts "# :method: #{expectation}"
    puts "# See Minitest::Assertions##{assertion}"
    puts
    puts "infect_an_assertion #{args.join ", "}"
  end
end

# vim: syntax=Ruby
