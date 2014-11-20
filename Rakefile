require "rake"

# Subscription folder names; ordered in reverse chronological order
# [{:name=>"56DySubscription_November_13__2014_-_January_08__2015", :from => Date, :to => Date}]
def subscriptions_name_from_to_dates
  names_dates = Dir["56Dy*"].map do |name|
    unless match = name.match(/56DySubscription_(?<from_month>.+)_(?<from_day>\d+)__(?<from_year>\d+)_-_(?<to_month>.+)_(?<to_day>\d+)__(?<to_year>\d+)/)
      $stderr.puts "Name `#{name}` does not match regular expression"
      exit 1
    end
    from = Date.parse("#{match[:from_day]} #{match[:from_month]} #{match[:from_year]}")
    to = Date.parse("#{match[:to_day]} #{match[:to_month]} #{match[:to_year]}")
    { name: name, from: from, to: to }
  end
  names_dates.sort {|nd1, nd2| nd2[:from] <=> nd1[:from] }
end

def pretty_print_airspaces(airspace_properties)
  airspace_properties.
    each do |prop|
      prop['AIRSPACE'] =~ /(.*), (.*)/
      prop['STATEFIRST_AIRSPACE'] = "#{$2}, #{$1}"
    end.
    sort {|p1, p2| p1['STATEFIRST_AIRSPACE'] <=> p2['STATEFIRST_AIRSPACE']}.
    each do |properties|
      puts "* #{properties['STATEFIRST_AIRSPACE']}, #{properties['NAME']}"
    end
end

namespace :shapefile do
  FileList['*/Additional_Data/Shape_Files/class_*.shp'].each do |src|
    geojson = src.gsub(/shp$/, "geo.json")
    file geojson => src do
      sh "shp2json #{src} -o #{geojson}"
    end
    task geojson: geojson

    topojson = src.gsub(/shp$/, "topo.json")
    file topojson => src do
      sh "topojson #{src} -o #{topojson}"
    end
    task topojson: topojson
  end

  desc "Diff latest JSON shapefiles"
  task :diff do
    require "json"
    require "json-compare"
    suffix = "geo.json"
    latest, previous, _ = subscriptions_name_from_to_dates
    latest_name, previous_name = latest[:name], previous[:name]
    %w[b c d].each do |airspace_class|
      latest_json, previous_json = [latest[:name], previous[:name]].map do |dirname|
        file = File.join(dirname, "Additional_Data/Shape_Files/class_#{airspace_class}.#{suffix}")
        JSON.parse(File.read(file))
      end
      diff = JsonCompare.get_diff(previous_json, latest_json)
      if diff.empty?
        puts "\nClass #{airspace_class.upcase}: no changes\n"
      else
        if (appended = diff[:update]["features"][:append]) && !appended.empty?
          puts "\nClass #{airspace_class.upcase}, additional:\n\n"
          airspace_properties = appended.map do |key, airspace|
            airspace['properties']
          end
          pretty_print_airspaces(airspace_properties)
        end
        if (updated = diff[:update]["features"][:update]) && !updated.empty?
          puts "\nClass #{airspace_class.upcase}, updated:\n\n"
          airspace_properties = updated.keys.inject([]) do |mem, updated_key|
            if previous = previous_json["features"][updated_key]
              mem << previous['properties']
            end
            mem
          end
          pretty_print_airspaces(airspace_properties)
        end
      end
    end
  end
end

desc "Convert .shp into JSON formats"
task shapefile: ["shapefile:geojson", "shapefile:topojson"]

namespace :fetch do
  desc "Fetch current NASR"
  task :latest do
    require "open-uri"
    require "nokogiri"
    domain = "https://nfdc.faa.gov"
    homepage = "#{domain}/xwiki/bin/view/NFDC/56+Day+NASR+Subscription"
    doc = Nokogiri::HTML(open(homepage))
    current_subscription_path = doc.at("//h2").parent.at("a").attributes["href"].value

    doc = Nokogiri::HTML(open("#{domain}#{current_subscription_path}"))
    download_path = doc.at("//p/a").attributes["href"].value

    subscription_file = File.basename(download_path)
    subscription_name = subscription_file.gsub(/.zip$/, "")
    unless File.directory?(subscription_name)
      download_url = "#{domain}#{download_path}"
      sh "curl #{download_url} -O"
      sh "unzip #{subscription_file} -d #{subscription_name}"
      sh "rm #{subscription_file}"
    end
  end
end

task default: ["fetch:latest", "shapefile"]
