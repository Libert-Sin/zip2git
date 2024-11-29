#!/usr/bin/env ruby

require 'fileutils'
require 'net/http'
require 'json'


zip_dir = 'zip파일 위치'
output_dir = '압축해제 위치'
organization = 'organization명'
github_token = '퍼스널 엑세스 토큰'


# 압축 해제 함수
def extract_zip(zip_file, output_dir, zip_dir)
  repo_name = File.basename(zip_file, '.zip')
  target_dir = File.join(output_dir, repo_name)

  # 압축 해제
  `unzip -q #{zip_file} -d #{output_dir}`

    target_dir
end


def initialize_git_repo(dir)
    Dir.chdir(dir) do
        `git init`
#        `git branch -m main`
        `git add .`
        `git commit -m "Initial commit"`
    end
end


def create_github_repo(repo_name, organization, github_token)
    url = URI("https://api.github.com/orgs/#{organization}/repos")
    req = Net::HTTP::Post.new(url)
    req['Authorization'] = "token #{github_token}"
    req['Content-Type'] = 'application/json'
    req.body = { name: repo_name, private: true }.to_json

    res = Net::HTTP.start(url.hostname, url.port, use_ssl: true) do |http|
        http.request(req)
    end

    if res.code.to_i == 201
        puts "GitHub repository '#{repo_name}' created successfully!"
        JSON.parse(res.body)['clone_url']
    else
        puts "Failed to create GitHub repository '#{repo_name}': #{res.body}"
        nil
    end
end


def push_to_github(dir, remote_url)
    Dir.chdir(dir) do
        `git remote add origin #{remote_url}`
        `git branch -M main`
        `git push -u origin main`
    end
end

# main
def process_zip_files(zip_dir, output_dir, organization, github_token)
    zip_files = Dir.glob(File.join(zip_dir, '*.zip'))
    FileUtils.mkdir_p(output_dir)

    zip_files.each do |zip_file|
        repo_name = File.basename(zip_file, '.zip')
        puts "Processing #{zip_file}..."

        # 1. zip
        repo_dir = extract_zip(zip_file, output_dir, zip_dir)
        puts "Extracted to #{repo_dir}"

        # 2. repi init
        initialize_git_repo(repo_dir)
        puts "Initialized Git repository at #{repo_dir}"

        # 3. Github make repo
        remote_url = create_github_repo(repo_name, organization, github_token)
        next unless remote_url

        # 4. Github Push
        push_to_github(repo_dir, remote_url)
        puts "Pushed to GitHub repository: #{remote_url}"
    end

    puts "All repositories processed successfully!"
end

# run
process_zip_files(zip_dir, output_dir, organization, github_token)
