require 'rake'
require 'fileutils'
require File.join(File.dirname(__FILE__), 'bin', 'yadr', 'neobundle')

desc "Hook our dotfiles into system-standard positions."
task :install => [:submodule_init, :submodules] do
  puts
  puts "======================================================"
  puts "Welcome to Magus Installation. (Powered by YADR)"
  puts "======================================================"
  puts

  install_homebrew if RUBY_PLATFORM.downcase.include?("darwin")
  install_rvm_binstubs

  # this has all the runcoms from this directory.
  if want_to_install?('git configs (color, aliases)')
    file_operation(Dir.glob('git/*'))
    run %{ git config --global core.excludesfile ~/.gitignore }
  end
  file_operation(Dir.glob('irb/*')) if want_to_install?('irb/pry configs (more colorful)')
  file_operation(Dir.glob('ruby/*')) if want_to_install?('rubygems config (faster/no docs)')
  run %{gem install tmuxinator html2haml} if want_to_install?('required gems') 
  file_operation(Dir.glob('ctags/*')) if want_to_install?('ctags config (better js/ruby support)')
  file_operation(Dir.glob('tmux/*')) if want_to_install?('tmux config')
  file_operation(Dir.glob('vimify/*')) if want_to_install?('vimification of command line tools')
  if want_to_install?('vim configuration (highly recommended)')
    file_operation(Dir.glob('{vim,vimrc}')) 
    Rake::Task["install_neobundle"].execute
  end

  Rake::Task["install_prezto"].execute

  install_fonts if RUBY_PLATFORM.downcase.include?("darwin")

  install_term_theme if RUBY_PLATFORM.downcase.include?("darwin")

  success_msg("installed")
end

task :install_prezto do
  if want_to_install?('zsh enhancements & prezto')
    install_prezto
  end
end

task :update do
  Rake::Task["neobundle_migration_from_pathogen"].execute if needs_migration_from_pathogen?
  Rake::Task["neobundle_migration_from_vundle"].execute if needs_migration_from_vundle?
  Rake::Task["install"].execute
  #TODO: for now, we do the same as install. But it would be nice
  #not to clobber zsh files
end

task :submodule_init do
  unless ENV["SKIP_SUBMODULES"]
    run %{ git submodule update --init --recursive }
  end
end

desc "Init and update submodules."
task :submodules do
  unless ENV["SKIP_SUBMODULES"]
    puts "======================================================"
    puts "Downloading Magus submodules...please wait"
    puts "======================================================"

    run %{
      cd $HOME/.magus
      git submodule update --recursive
      git clean -df
    }
    puts
  end
end

desc "Performs migration from pathogen to NeoBundle"
task :neobundle_migration_from_pathogen do
  puts "========================================================"
  puts "Migrating from pathogen to NeoBundle vim plugin manager."
  puts "This will move the old .vim/bundle directory to" 
  puts ".vim/bundle.old and replacing all your vim plugins with"
  puts "the standard set of plugins. You will then be able to "
  puts "manage your vim's plugin configuration by editing the "
  puts "file .vim/bundles.vim"
  puts "========================================================"

  Dir.glob(File.join('vim', 'bundle','**')) do |sub_path|
    run %{git config -f #{File.join('.git', 'config')} --remove-section submodule.#{sub_path}}
    # `git rm --cached #{sub_path}`
    FileUtils.rm_rf(File.join('.git', 'modules', sub_path))
  end
  FileUtils.mv(File.join('vim','bundle'), File.join('vim', 'bundle.old'))
end

desc "Performs migration from Vundle to NeoBundle"
task :neobundle_migration_from_vundle do
  puts "========================================================"
  puts "Migrating from Vundle to NeoBundle vim plugin manager."
  puts "This will delete everything inside .vim/bundle/ so"
  puts "NeoBundle can re-fetch them and manage them on its way."
  puts "You will then be able to manage your vim's plugin"
  puts "configuration by editing the file .vim/bundles.vim"
  puts "========================================================"

  FileUtils.rm_rf(Dir.glob(File.join('vim', 'bundle','*')))
end

desc "Runs NeoBundle installer in a clean vim environment"
task :install_neobundle do
  puts "======================================================"
  puts "Installing NeoBundle."
  puts "The installer will now proceed to run NeoBundleInstall."
  puts "Due to a bug, the installer may report some errors"
  puts "when installing the plugin 'syntastic'. Fortunately"
  puts "Syntastic will install and work properly despite the"
  puts "errors so please just ignore them and let's hope for"
  puts "an update that fixes the problem!"
  puts "======================================================"

  puts ""
  
  run %{
    cd $HOME/.magus
    git clone https://github.com/Shougo/neobundle.vim #{File.join('vim','bundle', 'neobundle.vim')}
  }

  NeoBundle::update_neobundle
end

task :default => 'install'


private
def run(cmd)
  puts "[Running] #{cmd}"
  `#{cmd}` unless ENV['DEBUG']
end

def install_rvm_binstubs
  puts "======================================================"
  puts "Installing RVM Bundler support. Never have to type"
  puts "bundle exec again! Please use bundle --binstubs and RVM"
  puts "will automatically use those bins after cd'ing into dir."
  puts "======================================================"
  run %{ chmod +x $rvm_path/hooks/after_cd_bundler }
  puts
end

def install_homebrew
  run %{which brew}
  unless $?.success?
    puts "======================================================"
    puts "Installing Homebrew, the OSX package manager...If it's"
    puts "already installed, this will do nothing."
    puts "======================================================"
    run %{ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"}
  end

  puts
  puts
  puts "======================================================"
  puts "Updating Homebrew."
  puts "======================================================"
  run %{brew update}
  puts
  puts
  puts "======================================================"
  puts "Installing Homebrew packages...There may be some warnings."
  puts "======================================================"
  run %{brew install zsh ctags git ghi hub tmux reattach-to-user-namespace the_silver_searcher fasd jslint}
  patch=`mvim --version | grep -n '1-\\d' | awk '{print $3}'`.gsub(/\d-/, "").to_i
  if patch <= 885
    run %{ brew uninstall macvim }
    run %{ brew install macvim --with-cscope --with-lua --HEAD }
  end
  puts
  puts
end

def install_fonts
  puts "======================================================"
  puts "Installing patched fonts for Powerline."
  puts "======================================================"
  run %{ cp -f $HOME/.magus/fonts/* $HOME/Library/Fonts }
  puts
end

def install_term_theme
  puts "======================================================"
  puts "Installing iTerm2 solarized theme."
  puts "======================================================"
  run %{ /usr/libexec/PlistBuddy -c "Add :'Custom Color Presets':'Solarized Light' dict" ~/Library/Preferences/com.googlecode.iterm2.plist }
  run %{ /usr/libexec/PlistBuddy -c "Merge 'iTerm2/Solarized Light.itermcolors' :'Custom Color Presets':'Solarized Light'" ~/Library/Preferences/com.googlecode.iterm2.plist }
  run %{ /usr/libexec/PlistBuddy -c "Add :'Custom Color Presets':'Solarized Dark' dict" ~/Library/Preferences/com.googlecode.iterm2.plist }
  run %{ /usr/libexec/PlistBuddy -c "Merge 'iTerm2/Solarized Dark.itermcolors' :'Custom Color Presets':'Solarized Dark'" ~/Library/Preferences/com.googlecode.iterm2.plist }

  # If iTerm2 is not installed or has never run, we can't autoinstall the profile since the plist is not there
  if !File.exists?(File.join(ENV['HOME'], '/Library/Preferences/com.googlecode.iterm2.plist'))
    puts "======================================================"
    puts "To make sure your profile is using the solarized theme"
    puts "Please check your settings under:"
    puts "Preferences> Profiles> [your profile]> Colors> Load Preset.."
    puts "======================================================"
    return
  end

  # Ask the user which theme he wants to install
  message = "Which theme would you like to apply to your iTerm2 profile?"
  color_scheme = ask message, iTerm_available_themes
  color_scheme_file = File.join('iTerm2', "#{color_scheme}.itermcolors")

  # Ask the user on which profile he wants to install the theme
  profiles = iTerm_profile_list
  message = "I've found #{profiles.size} #{profiles.size>1 ? 'profiles': 'profile'} on your iTerm2 configuration, which one would you like to apply the Solarized theme to?"
  profiles << 'All'
  selected = ask message, profiles
  
  if selected == 'All'
    (profiles.size-1).times { |idx| apply_theme_to_iterm_profile_idx idx, color_scheme_file }
  else
    apply_theme_to_iterm_profile_idx profiles.index(selected), color_scheme_file
  end
end

def iTerm_available_themes
   Dir['iTerm2/*.itermcolors'].map { |value| File.basename(value, '.itermcolors')}
end

def iTerm_profile_list
  profiles=Array.new
  begin
    profiles <<  %x{ /usr/libexec/PlistBuddy -c "Print :'New Bookmarks':#{profiles.size}:Name" ~/Library/Preferences/com.googlecode.iterm2.plist 2>/dev/null}
  end while $?.exitstatus==0
  profiles.pop
  profiles
end

def ask(message, values)
  puts message
  while true
    values.each_with_index { |val, idx| puts " #{idx+1}. #{val}" }
    selection = STDIN.gets.chomp
    if (Float(selection)==nil rescue true) || selection.to_i < 0 || selection.to_i > values.size+1
      puts "ERROR: Invalid selection.\n\n"
    else
      break
    end
  end 
  selection = selection.to_i-1
  values[selection]
end

def install_prezto
  puts
  puts "Installing Prezto (ZSH Enhancements)..."

  unless File.exists?(File.join(ENV['ZDOTDIR'] || ENV['HOME'], ".zprezto"))
    run %{ ln -nfs "$HOME/.magus/zsh/prezto" "${ZDOTDIR:-$HOME}/.zprezto" }

    # The prezto runcoms are only going to be installed if zprezto has never been installed
    file_operation(Dir.glob('zsh/prezto/runcoms/z*'), :copy)
  end

  puts
  puts "Overriding prezto ~/.zpreztorc with Magus's zpreztorc to enable additional modules..."
  run %{ ln -nfs "$HOME/.magus/zsh/prezto-override/zpreztorc" "${ZDOTDIR:-$HOME}/.zpreztorc" }

  puts
  puts "Creating directories for your customizations"
  run %{ mkdir -p $HOME/.zsh.before }
  run %{ mkdir -p $HOME/.zsh.after }
  run %{ mkdir -p $HOME/.zsh.prompts }

  if ENV["SHELL"].include? 'zsh' then
    puts "Zsh is already configured as your shell of choice. Restart your session to load the new settings"
  else
    puts "Setting zsh as your default shell"
    run %{ chsh -s /bin/zsh }
  end
end

def want_to_install? (section)
  if ENV["ASK"]=="true"
    puts "Would you like to install configuration files for: #{section}? [y]es, [n]o"
    STDIN.gets.chomp == 'y'
  else
    true
  end
end

def file_operation(files, method = :symlink)
  files.each do |f|
    file = f.split('/').last
    source = "#{ENV["PWD"]}/#{f}"
    target = "#{ENV["HOME"]}/.#{file}"

    puts "======================#{file}=============================="
    puts "Source: #{source}"
    puts "Target: #{target}"

    if File.exists?(target) && (!File.symlink?(target) || (File.symlink?(target) && File.readlink(target) != source))
      puts "[Overwriting] #{target}...leaving original at #{target}.backup..."
      run %{ mv "$HOME/.#{file}" "$HOME/.#{file}.backup" }
    end

    if method == :symlink
      run %{ ln -nfs "#{source}" "#{target}" }
    else
      run %{ cp -f "#{source}" "#{target}" }
    end

    # Temporary solution until we find a way to allow customization
    # This modifies zshrc to load all of Magus's zsh extensions.
    # Eventually Magus's zsh extensions should be ported to prezto modules.
    if file == 'zshrc'
      File.open(target, 'a') do |zshrc|
        zshrc.puts('for config_file ($HOME/.magus/zsh/*.zsh) source $config_file')
      end
    end

    puts "=========================================================="
    puts
  end
end

def needs_migration_from_pathogen?
  File.exists? File.join('vim', 'bundle', 'tpope-vim-pathogen')
end

def needs_migration_from_vundle?
  File.exists? File.join('vim', 'vundles.vim')
end

def list_vim_submodules
  result=`git submodule -q foreach 'echo $name"||"\`git remote -v | awk "END{print \\\\\$2}"\`'`.select{ |line| line =~ /^vim.bundle/ }.map{ |line| line.split('||') }
  Hash[*result.flatten]
end

def apply_theme_to_iterm_profile_idx(index, color_scheme_path)
  values = Array.new
  16.times { |i| values << "Ansi #{i} Color" }
  values << ['Background Color', 'Bold Color', 'Cursor Color', 'Cursor Text Color', 'Foreground Color', 'Selected Text Color', 'Selection Color']
  values.flatten.each { |entry| run %{ /usr/libexec/PlistBuddy -c "Delete :'New Bookmarks':#{index}:'#{entry}'" ~/Library/Preferences/com.googlecode.iterm2.plist } }

  run %{ /usr/libexec/PlistBuddy -c "Merge '#{color_scheme_path}' :'New Bookmarks':#{index}" ~/Library/Preferences/com.googlecode.iterm2.plist }
end

def success_msg(action)
  puts ""
  puts "`MMb     dMM'                                     "
  puts " MMM.   ,PMM   Like Janus, but with more Magic!   "
  puts " M`Mb   d'MM    ___     __     ___   ___   ____   "
  puts " M YM. ,P MM  6MMMMb   6MMbMMM `MM    MM  6MMMMb\ "
  puts " M `Mb d' MM 8M'  `Mb 6M'`Mb    MM    MM MM'    ` "
  puts " M  YM.P  MM     ,oMM MM  MM    MM    MM YM.      "
  puts " M  `Mb'  MM ,6MM9'MM YM.,M9    MM    MM  YMMMMb  "
  puts " M   YP   MM MM'   MM  YMM9     MM    MM      `Mb "
  puts " M   `'   MM MM.  ,MM (M        YM.   MM L    ,MM "
  puts "_M_      _MM_`YMMM9'Yb.YMMMMb.   YMMM9MM_MYMMMM9  "
  puts "                      6M    Yb                    "
  puts "                      YM.   d9   Powered by YADR  "
  puts "                       YMMMM9                     " 
  puts ""
  puts "Magus has been #{action}. Please restart your terminal and vim."
end
