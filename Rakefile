require 'json'
require 'aws-sdk'
require 'digest/md5'
require 'mime/types'
require 'highline/import'
require 'colorize'

class S3Client
  def open_s3_connection
    begin
      return AWS::S3.new aws_credentials
    rescue Errno::ENOENT
      puts "Credentials file not found. Please run 's3:auth' to authenticate."
      abort
    end
  end

  def authenticate
    if confirm_create_credentials_file?
      request_credentials
      create_credentials_file!
    end
  end

  private

  def aws_credentials
    JSON.parse File.read credentials_path
  end

  def initialize
    @credentials = {}
  end

  def create_credentials_file!
    File.open credentials_path, "w" do |f|
      f.write @credentials.to_json
    end
  end

  def request_credentials
    puts "Please enter your AWS access key id."
    @credentials[:access_key_id] = STDIN.gets.chomp
    access_key = ask "Please enter your AWS secret access key" do |question|
      question.echo = false
    end
    @credentials[:secret_access_key] = access_key
  end

  def credentials_path
    File.expand_path "~/.aws.json"
  end

  def confirm_create_credentials_file?
    if File.exists? credentials_path
      puts "Override existing '.aws.json' file? (y/n)"
      answer = STDIN.gets.chomp
      answer == "y"
    else
      true
    end
  end
end

class S3Upload
  def sync path
    puts "Uploading files to bucket '#{ bucket.name }'"
    walk_and_upload path
    puts "Done syncing bucket"
  end

  def set_bucket
    unless bucket.exists?
      puts "Creating bucket '#{ bucket.name }'"
      s3.buckets.create(bucket.name, acl: :bucket_owner_full_control)
    end
  end

  private

  attr_reader :s3, :bucket

  def initialize s3_client
    @s3 = s3_client

    bucket_name = ENV['bucket'] || File.basename(Dir.getwd)
    @bucket = @s3.buckets[bucket_name]
    set_bucket
  end

  def walk_and_upload path
    entries = Dir.entries path
    entries.each do |entry|
      next if entry == File.basename(__FILE__) || entry[0] == '.' 
      nested_entry = (path == "." ? entry : "#{ path }/#{ entry }")
      if File.directory? nested_entry
        walk_and_upload nested_entry
        next
      else
        upload nested_entry
      end
    end
  end

  def upload entry
    new_object = bucket.objects[entry]

    begin
      if new_object.exists?
        # Strip opening and closing "\" chars from AWS-formatted etag
        etag_is_same = new_object.etag[1..-2] == Digest::MD5.hexdigest(File.read entry)

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
      new_object.write File.open entry, content_type: content_type
    rescue AWS::S3::Errors::Forbidden
      puts "Access denied!"
      print "Make sure your credentials are correct "
      print "and your bucket name isn't already taken by someone else."
      puts
      print "Note: AWS bucket names are shared across all users."
      puts
      abort
    end
  end
end

namespace :s3 do
  desc "Deploy all files to S3"
  task :upload do
    s3_client = S3Client.new
    s3_connection = s3_client.open_s3_connection
    s3_upload = S3Upload.new s3_connection

    STDOUT.sync = true # Show progress
    s3_upload.sync "."
    STDOUT.sync = false # Done with progress output.
  end

  task :auth do
    s3_client = S3Client.new
    s3_client.authenticate
  end
end

