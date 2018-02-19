PROJECT_NAME  = PROJECT_SLUG
SASS_FOLDER   = 'wp-content/themes/$(PROJECT_NAME)/sass'
JS_FOLDER     = 'wp-content/themes/$(PROJECT_NAME)/javascript'
STATIC_FOLDER = 'wp-content/themes/$(PROJECT_NAME)/static'

OS 						:= $(shell uname)
SHELL       	:= $(shell which bash)
.SHELLFLAGS 	= -c

WEBPACK_PORT             = 2992
SLACK_HOOK 		           = ''
SENTRY_TOKEN 	           = ''
TIMESTAMP 		           = `date +%s`
BRANCH 				           = $(shell git rev-parse --abbrev-ref HEAD)
GIT_SHA 			           = $(shell git rev-parse HEAD)

IP_PRODUCTION = ''
IP 						= ''
USER 					= 'jackjoe'

T_STAGING 	 	= '/var/www/staging.tenderwolf.com'
T_PRODUCTION 	= '/var/www/tenderwolf.com'

.SILENT: ;               # no need for @
.ONESHELL: ;             # recipes execute in same shell
.NOTPARALLEL: ;          # wait for this target to finish
.EXPORT_ALL_VARIABLES: ; # send all vars to shell
Makefile: ;              # skip prerequisite discovery

.PHONY: run restart app flow sass deploy production build test uglify yaml_lint

sass: ## compile sass
	echo "sass compile .."
	node_modules/node-sass/bin/node-sass \
		-o $(STATIC_FOLDER)/css \
		--output-style compressed \
		--source-map true \
		$(SASS_FOLDER)/style.sass
	node_modules/postcss-cli/bin/postcss \
		$(STATIC_FOLDER)/css/style.css \
		--use autoprefixer \
		--replace

ifeq ($(OS), Darwin)
	node_modules/node-notifier/vendor/terminal-notifier.app/Contents/MacOS/terminal-notifier \
	  -message "Sass Compiled"
endif

sasswatch: ## compile sass
	echo 'Watching sass...'
	node_modules/node-sass/bin/node-sass \
		-w $(SASS_FOLDER) \
		-r \
		-o $(STATIC_FOLDER)/css \
		--output-style compressed \
		--sourcemap true \
		$(SASS_FOLDER)/style.sass

ifeq ($(OS), Darwin)
	node_modules/node-notifier/vendor/terminal-notifier.app/Contents/MacOS/terminal-notifier \
	  -message "Sass Compiled"
endif

app:
	node node_modules/webpack/bin/webpack.js \
					--progress \
					--colors

ifeq ($(OS), Darwin)
	node_modules/node-notifier/vendor/terminal-notifier.app/Contents/MacOS/terminal-notifier \
					-message "App Compiled"
endif

####################

build: sass app

production_clean:
	echo "Removing and reinstalling node_modules"
	rm -rf ./node_modules
	yarn

production: sass
	node_modules/postcss-cli/bin/postcss \
					$(STATIC_FOLDER)/css/style.css \
					--config postcss.config.js \
					--replace
	NODE_ENV=production GIT_SHA=$(GIT_SHA) SENTRY_TOKEN=$(SENTRY_TOKEN) node node_modules/webpack/bin/webpack.js \
					--progress \
					--colors
	echo "/* Jack + Joe -" `date "+%F +%T"` "*/" | \
					cat - $(STATIC_FOLDER)/javascript/app.min.js > \
					temp && mv temp public/javascript/app.min.js
ifeq ($(OS), Darwin)
	node_modules/node-notifier/vendor/terminal-notifier.app/Contents/MacOS/terminal-notifier \
								-message "Production Build"
endif

####################

server:
	NODE_ENV=local node node_modules/webpack-dev-server/bin/webpack-dev-server.js \
					--config webpack.config.js \
					--progress \
					--colors \
					--port $(WEBPACK_PORT)

analyse_app:
	NODE_ENV=production ANALYSE=true node node_modules/webpack/bin/webpack.js \
					--progress \
					--colors

####################

# deploy_staging_files: _deploy_staging
# deploy_staging: production _deploy_staging
# _deploy_staging:
# 	@ssh $(USER)@$(IP) '\
# 					cd $(T_STAGING) ;\
# 					rm -rf vendor ;\
# 					sudo chmod -R 775 . ;\
# 					'
# 	@rsync \
# 					-vrzc \
# 					--delete \
# 					--no-p \
# 					--no-o \
# 					--exclude-from=./.deployignore \
# 					. \
# 					$(USER)@$(IP):$(T_STAGING)
# 	@ssh $(USER)@$(IP) '\
# 					cd $(T_STAGING) ;\
# 					rm -f .env ;\
# 					cp .env.staging .env ;\
# 					sed -i.bak -e "s/ASSET_VERSION=.*/ASSET_VERSION=$(GIT_SHA)/g" .env ;\
# 					'
# 	@make compose_staging
# 	@make finish_staging
# 	@make notify_staging
#
# compose_staging:
# 	@ssh $(USER)@$(IP) '\
# 					cd $(T_STAGING) ;\
# 					mkdir -p storage/framework ;\
# 					mkdir -p storage/framework/cache ;\
# 					mkdir -p storage/framework/sessions ;\
# 					mkdir -p storage/framework/views ;\
# 					sudo chmod -R 775 public ;\
# 					sudo chmod -R 777 storage ;\
# 					sudo chown -R www-data:www-data storage ;\
# 					sudo /bin/bash -c "rm -f storage/logs/*.log" ;\
# 					rm -rf vendor ;\
# 					composer install --no-dev --no-suggest --no-progress ;\
# 					'
#
# finish_staging:
# 	@ssh $(USER)@$(IP) '\
# 					cd $(T_STAGING) ;\
# 					php artisan clear-compiled ;\
# 					php artisan config:clear ;\
# 					php artisan cache:clear ;\
# 					php artisan migrate ;\
# 					php artisan db:seed ;\
# 					php artisan opcache:clear; \
# 					'
#
# notify_staging:
# 	@curl -X POST \
# 					--data-urlencode 'payload={"channel": "#deploy-bot", "username": "$(USER)", "text": "Tenderwolf deployed to staging", "icon_emoji": ":wolf:"}' \
# 					$(SLACK_HOOK)
