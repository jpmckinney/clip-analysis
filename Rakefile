# coding: utf-8
require 'rubygems'
require 'bundler/setup'

desc 'Download all Canadian licenses from CLIP'
task :download do
  require 'fileutils'
  require 'open-uri'
  require 'psych'

  require 'nokogiri'
  require 'unicode_utils/downcase'

  BASE_URL = 'http://clip.cippic.ca/'

  class String
    def slug
      UnicodeUtils.downcase(gsub(/[[:space:]—–-]+/, ' ').strip, :fr).gsub(/\p{Punct}|\p{Cntrl}/, '').split.join('-').tr(
        "ÀÁÂÃÄÅàáâãäåĀāĂăĄąÇçĆćĈĉĊċČčÐðĎďĐđÈÉÊËèéêëĒēĔĕĖėĘęĚěĜĝĞğĠġĢģĤĥĦħÌÍÎÏìíîïĨĩĪīĬĭĮįİıĴĵĶķĸĹĺĻļĽľĿŀŁłÑñŃńŅņŇňŉŊŋÒÓÔÕÖØòóôõöøŌōŎŏŐőŔŕŖŗŘřŚśŜŝŞşŠšſŢţŤťŦŧÙÚÛÜùúûüŨũŪūŬŭŮůŰűŲųŴŵÝýÿŶŷŸŹźŻżŽž",
        "aaaaaaaaaaaaaaaaaaccccccccccddddddeeeeeeeeeeeeeeeeeegggggggghhhhiiiiiiiiiiiiiiiiiijjkkkllllllllllnnnnnnnnnnnoooooooooooooooooorrrrrrsssssssssttttttuuuuuuuuuuuuuuuuuuuuwwyyyyyyzzzzzz")
    end
  end

  FileUtils.mkdir 'data'

  Nokogiri::HTML(open("#{BASE_URL}license-list.php?cat=Canadian")).css('.license-list li').each do |li|
    id    = li.at_css('.lid').text
    title = li.at_css('a').text
    slug  = title.slug

    File.open("data/#{slug}.yml", 'w') do |f|
      f.write Psych.dump id: id, title: title, maintainer: li.at_css('.lmaint').text
    end

    File.open("data/#{slug}.txt", 'w') do |f|
      f.write Nokogiri::HTML(open("#{BASE_URL}license-text.php?id=#{id}")).at_css('#license-text-wrap').text
    end
  end
end