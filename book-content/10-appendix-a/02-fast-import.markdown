	#! /usr/bin/env ruby
	require 'digest/sha1'

	$author = 'Scott Chacon <schacon@geemail.com>'
	$marks = []

	def convert_mark_to_date(mark)
	  if mark == 'current'
	    return Time.now().to_i
	  else
	    mark = mark.gsub('back_', '')
	    (year, month, day) = mark.split('_')
	    return Time.local(year, month, day).to_i
	  end
	end

	def export_data(string)
	  puts "data <<EOF"
	  puts string
	  puts 'EOF'
	end

	def inline_data(file, code = 'M', mode = '644')
	  content = File.read(file)
	  puts "#{code} #{mode} inline #{file}"
	  export_data(content)
	end

	def convert_dir_to_mark(dir)
	  if !$marks.include?(dir)
	    $marks << dir
	  end
	  ($marks.index(dir) + 1).to_s
	end

	def print_export(dir, last_mark, last_manifest)
  
	  mark = convert_dir_to_mark(dir)
	  date = convert_mark_to_date(dir)
  
	  # print the import information
	  puts 'commit refs/heads/master'
	  puts 'mark :' + mark
	  puts "committer #{$author} #{date} -0700"
	  export_data('imported from ' + dir)
	  puts 'from :' + last_mark if last_mark

	  current_manifest = {}
	  Dir.glob("**/*").each do |dir|
	    next if !File.file?(dir)
	    sha = Digest::SHA1.file(dir).hexdigest
	    current_manifest[dir] = sha
	  end
  
	  # add new files
	  new_files = current_manifest.keys - last_manifest.keys
	  new_files.each do |newf|
	    inline_data(newf)
	  end
  
	  # remove deleted files
	  removed_files = last_manifest.keys - current_manifest.keys
	  removed_files.each do |del|
	    puts "D #{del}"
	  end
  
	  # modify changed files
	  current_manifest.each do |filenm, sha| 
	    next if !last_manifest.has_key?(filenm)
	    if last_manifest[filenm] != sha
	      inline_data(filenm)
	    end
	  end

	  return [mark, current_manifest]
	end


	last_manifest = {}
	last_mark = nil

	# loop through the directories
	Dir.chdir(ARGV[0]) do
	  Dir.glob("*").each do |dir|
	    next if File.file?(dir)

	    # move into the target directory
	    Dir.chdir(dir) do 
	      (last_mark, last_manifest) = print_export(dir, last_mark, last_manifest)
	    end
	  end
	end