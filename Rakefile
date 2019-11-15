## --- BEGIN LICENSE BLOCK ---
# Copyright (c) 2017-present WeWantToKnow AS
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
## --- END LICENSE BLOCK ---

require 'u3d'
require 'json'
require 'fileutils'
UI = U3dCore::UI
VERSION = '1'

# RuboCop::RakeTask.new

# update cache and return cache path
def update_cache
  U3dCore::Globals.verbose = true
  U3dCore::Helper.operating_systems.each do |os|
    U3d::Cache.new(force_os: os, force_refresh: true)
  end
  path = File.expand_path('cache.json', U3d::Cache.default_os_path)

  public_cache_name = 'versions.json'
  version_directory =  'v' + VERSION

  FileUtils.mkdir_p version_directory
  public_cache_path = File.join(version_directory, public_cache_name)

  current_cache = File.exist?(public_cache_path) ? File.read(public_cache_path) : '{}'
  new_cache = File.read(path)

  if JSON.parse(ignore_last_update(current_cache)) != JSON.parse(ignore_last_update(new_cache))
    UI.message('Cache data updated')
  end

  File.write(public_cache_path, new_cache)
end

def ignore_last_update(s)
  s.gsub(/(\"lastupdate\":)[0-9]+,/, '"lastupdate":0,')
end

def run_command(command, error_message = nil)
  output = `#{command}`
  unless $CHILD_STATUS.success?
    error_message = "Failed to run command '#{command}'" if error_message.nil?
    UI.user_error!(error_message)
  end
  output
end

task :test_update do
  update_cache
end

task :update do
  sh 'git checkout gh-pages'
  update_cache
  sh "git add v#{VERSION}/versions.json"
  sh 'git config --global user.email "ci@dragonbox.com"' if `git config --global user.email`.empty?
  sh 'git config --global user.name "CI"' if `git config --global user.name`.empty?
  sh 'git commit -m "Automated cache update [ci skip]"'
  sh 'git push origin gh-pages'
end

task default: %i[update]
