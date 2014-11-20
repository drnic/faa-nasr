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
