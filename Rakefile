require "rake"

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
