
require 'haml'
require 'sass'

def gen(engine, in_name, out_name, options = {})
  Dir["**/*#{in_name}"].each do |filename|
    out_file = filename.sub(/#{in_name}$/, out_name)
    open(filename) do |content|
      open(out_file, "w") do |output|
        output.print engine.new(content.read, options).render
      end
    end
  end
end

task :haml do 
  gen(Haml::Engine, ".haml", ".html", :format => :html5)
end

task :sass do 
  gen(Sass::Engine, ".sass", ".css")
end

namespace :gen do
  task :all => [:haml, :sass]
end
