# codelog.luzi82.com

## Local preview

### Init env

    # Base on https://docs.github.com/en/github/working-with-github-pages/testing-your-github-pages-site-locally-with-jekyll
    # Assume Debian 10
    
    sudo apt-get install ruby-full build-essential
    
    echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
    echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
    echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    
    gem install jekyll bundler
    
    cd ${PROJECT_ROOT}/docs
    bundle install

### Preview

    cd ${PROJECT_ROOT}
    ./preview.sh

Preview in http://127.0.0.1:4000
