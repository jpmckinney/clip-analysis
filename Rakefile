# coding: utf-8
require 'rubygems'
require 'bundler/setup'

require 'psych'
require 'ruby-progressbar'

# @todo The text in these documents could be cleaner. Go through the DOM,
#   transforming the HTML to Markdown? https://github.com/29decibel/html2markdown
desc 'Download all Canadian licenses from CLIP'
task :download do
  require 'fileutils'
  require 'open-uri'

  require 'nokogiri'
  require 'unicode_utils/downcase'

  BASE_URL = 'http://clip.cippic.ca/'
  FileUtils.mkdir_p 'data'

  lis = Nokogiri::HTML(open("#{BASE_URL}license-list.php?cat=Canadian")).css('.license-list li')
  progressbar = ProgressBar.create format: '%a |%B| %p%% %e', length: 80, smoothing: 0.5, total: lis.size

  lis.each do |li|
    progressbar.increment

    id    = li.at_css('.lid').text
    title = li.at_css('a').text
    text  = Nokogiri::HTML(open("#{BASE_URL}license-text.php?id=#{id}")).at_css('#license-text-wrap').text

    File.open("data/#{id}.yml", 'w') do |f|
      f.write Psych.dump id: id, title: title, maintainer: li.at_css('.lmaint').text
    end

    File.open("data/#{id}.txt", 'w') do |f|
      f.write text.split("\n").map(&:strip).join("\n").strip
    end
  end
end

namespace :similarity do
  desc 'Create a similarity matrix, using tf*idf as the similarity measure'
  task :tfidf do
    require 'tf-idf-similarity'

    licenses = Dir['data/*.txt']
    progressbar = ProgressBar.create format: '%a |%B| %p%% %e', length: 80, smoothing: 0.5, total: licenses.size

    corpus = TfIdfSimilarity::Collection.new
    licenses.each do |license|
      progressbar.increment
      corpus << TfIdfSimilarity::Document.new(File.read(license))
    end
    similarity_matrix = corpus.similarity_matrix

    matrix = {}
    licenses.each_with_index do |a,i|
      a_id = File.basename a, '.txt'
      matrix[a_id] = {}
      licenses.each_with_index do |b,j|
        b_id = File.basename b, '.txt'
        matrix[a_id][b_id] = similarity_matrix[i, j].round(6)
      end
    end

    File.open('similarity-tfidf.yml', 'w') do |f|
      f.write Psych.dump matrix
    end
  end

  # Diff only counts insertion and deletion operations. Damerau–Levenshtein
  # distance also counts substitution and transposition. However, in our
  # texts, the latter two operations are rare.
  desc 'Create a similarity matrix, using diff as the similarity measure'
  task :diff do
    require 'diff_match_patch_native'

    class Fixnum
      # @return [Integer] the factorial of the number
      # @note This implementation only works for numbers greater than one.
      def factorial
        (2..self).reduce(:*)
      end

      # @param [Integer] k the size of a combination
      # @return [Integer] the number of k-combinations of a set of n elements
      def combinations(k)
        self.factorial / (k.factorial * (self - k).factorial)
      end
    end

    licenses = Dir['data/*.txt']
    progressbar = ProgressBar.create format: '%a |%B| %p%% %e', length: 80, smoothing: 0.5, total: licenses.size.combinations(2)

    matrix = {}
    dmp = DiffMatchPatch.new

    licenses.each_with_index do |a,i|
      a_id   = File.basename a, '.txt'
      a_text = File.read a

      licenses[i + 1..-1].each do |b|
        b_id   = File.basename b, '.txt'
        b_text = File.read b

        progressbar.increment

        # Count common characters.
        summary = Hash.new 0
        dmp.diff_main(a_text, b_text, false).each do |operation,string|
          summary[operation] += string.size
        end

        # http://en.wikipedia.org/wiki/Dice%27s_coefficient
        similarity = 2 * summary[0] / (a_text.size + b_text.size).to_f

        # Build the similarity matrix,
        matrix[a_id] ||= {a_id => 1}
        matrix[a_id][b_id] = similarity
        matrix[b_id] ||= {b_id => 1}
        matrix[b_id][a_id] = similarity
      end
    end

    File.open('similarity-diff.yml', 'w') do |f|
      f.write Psych.dump matrix
    end
  end
end

namespace :visualize do
  desc 'Print a similarity matrix'
  task :print, :file do |t,args|
    require 'colored'

    Psych.load(File.read(args[:file])).sort_by{|k,_| k}.each_with_index do |(id,row),index|
      row = row.sort_by{|k,_| k}

      if index == 0
        print ' ' * 16
        row.each do |k,_|
          print k.split('-')[1][0..2].ljust(4)
        end
        puts
      end

      print id.ljust(15)
      row.each do |_,value|
        string = ('%d' % (value * 100)).rjust(4)
        if value > 0.7
          print string.green
        else
          print string
        end
      end
      puts
    end
  end

  # http://en.wikipedia.org/wiki/Dendrogram
  # http://en.wikipedia.org/wiki/Hierarchical_clustering
  desc 'Draw a dendrogram with D3'
  task :dendrogram, :file do |t,args|
    require 'rbcluster'

    data = []
    Psych.load(File.read(args[:file])).each do |id,row|
      data << row.values
    end

    # http://bonsai.hgc.jp/~mdehoon/software/cluster/cluster.pdf
    # @todo rbcluster doesn't implement passing distancematrix as an option.
    tree = Cluster.treecluster([[]], distancematrix: data)
    tree.scale

    # @todo Transform the output into a dendrogram using D3
    # https://github.com/mbostock/d3/wiki/Cluster-Layout
  end

  # http://en.wikipedia.org/wiki/Dendrogram
  # http://en.wikipedia.org/wiki/Hierarchical_clustering
  desc 'Build an HTML interface for comparing documents'
  task :diff, :file do |t,args|
    # @todo
  end
end
