require 'pry'
require 'nokogiri'
require 'open-uri'

def fetch_detailed_information(url)

  find_value = ->(e, key) {
    e.at_xpath("//div[span[@class='event-header-selector' and text() = '#{key}']]/span[@class='event-content-field']").text.strip
  }

  doc = Nokogiri::HTML(open(url))
  result = {}
  doc.css('#event-information').each do |e|
    result = {
      start_date: find_value.call(e, 'Start'),
      end_date: find_value.call(e, 'End'),
      times: find_value.call(e, 'Times'),
      venue: find_value.call(e,'Venue'),
      address: find_value.call(e, 'Address'),
      tel: find_value.call(e, 'Telephone'),
      web_url: find_value.call(e, 'Website'),
      email: find_value.call(e, 'Email'),
      cost: find_value.call(e, 'Cost')
    }
  end

  result[:preview_image] = "http://www.artlyst.com/#{doc.css('.view-page-image img').at_xpath('@src').text}"

  result
end

def fetch_exhibition_list(name)
    url = "http://www.artlyst.com/#{name}"

    doc = Nokogiri::HTML(open(url))

    serverdate = doc.css('.serverdate').first.text.gsub("\u00A0", "").sub('|','').strip

    data = []
    doc.css('.featuredevents #featuredevents-listings').each do |e|
      content = e.css('.featured-listings-content')
      title = content.css('.event-name').first.text.strip
      link = content.css('a').first.at_xpath('@href')
      description = content.css('p').first.text.strip

      params = fetch_detailed_information("http://www.artlyst.com#{link}")
      params[:title] = title
      params[:description] = description

      data.push(params)
    end
    generate_xml(data, name, serverdate)
end

def generate_xml(data, filename, serverdate)
  builder = Nokogiri::XML::Builder.new do |xml|
    xml.exhibitions {
      xml.serverdate serverdate
      data.each do |e|
        xml.exhibition {
          xml.title e[:title]
          xml.start_date e[:start_date]
          xml.end_date e[:end_date]
          xml.times e[:times]
          xml.venue e[:venue]
          xml.address e[:address]
          xml.web_url e[:web_url]
          xml.email e[:email]
          xml.cost e[:cost]
          xml.preview_image e[:preview_image]
        }
      end
    }
  end

  puts "Write to #{ filename }.xml"
  File.open("#{ filename }.xml",'w') { |f| f.write builder.to_xml }
end

namespace :data do

  task :artlyst_emerging do
    fetch_exhibition_list('emerging-exhibitions')
  end

  task :artlyst_featured do
    fetch_exhibition_list('featured-exhibitions')
  end

end
