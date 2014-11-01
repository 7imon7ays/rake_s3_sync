require 'aws-sdk'
require 'digest/md5'
require 'mime/types'
require 'colorize'

AWS_ACCESS_KEY_ID = "XXXX"
AWS_SECRET_ACCESS_KEY = "YYYY"
BUCKET_NAME = "ZZZZ"

class S3Connection
  attr_reader :s3, :bucket

  def sync path
    puts "Uploading files to bucket '#{ bucket.name }'"
    walk_and_upload path
    puts "Done syncing bucket"
  end

  def set_bucket
    unless bucket.exists?
      puts "== Creating bucket '#{ BUCKET_NAME }'"
      s3.buckets.create(BUCKET_NAME, acl: :bucket_owner_full_control)
    end
  end

  private

  def initialize
    @s3 = AWS::S3.new(
      access_key_id: AWS_ACCESS_KEY_ID,
      secret_access_key: AWS_SECRET_ACCESS_KEY
    )

    @bucket = @s3.buckets[BUCKET_NAME]
  end

  def walk_and_upload path
    entries = Dir.entries path
    entries.each do |entry|
      next if entry[0] == '.'
      nested_entry = (path == "." ? entry : "#{ path }/#{ entry }")
      if File.directory?(nested_entry)
        walk_and_upload nested_entry
        next
      else
        upload nested_entry
      end
    end
  end

  def upload entry
    @new_object = bucket.objects[entry]

    if @new_object.exists?
      # Strip opening and closing "\" chars from AWS-formatted etag
      etag_is_same = @new_object.etag[1..-2] == Digest::MD5.hexdigest(File.read entry)

      if etag_is_same
        puts "\tUnchanged: #{ entry }".blue
        return
      else
        puts "\tUpdating: '#{ entry }'".yellow
      end
    else
      puts "\tUploading: '#{ entry }'".green
    end
    content_type = MIME::Types.type_for(entry).to_s
    @new_object.write(File.open entry, content_type: content_type)
  end
end

namespace :s3 do
  desc "Deploy all files to S3"
  task :upload, :env, :branch do |t, args|
    s3_connection = S3Connection.new
    s3_connection.set_bucket

    STDOUT.sync = true # Show progress
    s3_connection.sync(".")
    STDOUT.sync = false # Done with progress output.
  end
end
