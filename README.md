# Using hooks to deploy jekyll site

## Install ruby:

	yum update
	yum install gcc-c++ patch readline readline-devel zlib zlib-devel 
	yum install libyaml-devel libffi-devel openssl-devel make 
	yum install bzip2 autoconf automake libtool bison iconv-devel
	curl -L get.rvm.io | bash -s stable
	source /etc/profile.d/rvm.sh
	rvm install 1.9.3
	rvm use 1.9.3 --default 
	ruby --version

See [http://tecadmin.net/install-ruby-1-9-3-or-multiple-ruby-verson-on-centos-6-3-using-rvm/](http://tecadmin.net/install-ruby-1-9-3-or-multiple-ruby-verson-on-centos-6-3-using-rvm/)


## Install nodejs:

	curl -sL https://rpm.nodesource.com/setup | bash -
	yum install -y nodejs

See [https://github.com/joyent/node/wiki/installing-node.js-via-package-manager](https://github.com/joyent/node/wiki/installing-node.js-via-package-manager)

## Setup hooks

On server:

	mkdir ~/repo/gibandjo.com.git
	cd ~/repo/gibandjo.com.git
	git --bare init
	cat > hooks/post-receive
	GIT_REPO=$HOME/repo/gibandjo.com.git
	TMP_GIT_CLONE=$HOME/tmp/gibandjo.com
	PUBLIC_WWW=/var/www/gibandjo.com/public_html

	git clone $GIT_REPO $TMP_GIT_CLONE
	jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
	rm -Rf $TMP_GIT_CLONE
	exit

Then press `ctrl+d`

On local:

	git remote add deploy root@explodecomputer.com:~/repo/gibandjo.com.git
	git push deploy master

See [http://jekyllrb.com/docs/deployment-methods/](http://jekyllrb.com/docs/deployment-methods/)

## Using Rake

On local:

	cd ~/repo/gibandjo.com
	rake deploy

See [https://github.com/avillafiorita/jekyll-rakefile](https://github.com/avillafiorita/jekyll-rakefile)
