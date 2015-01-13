export WERCKER_FTP_DEPLOY_DESTINATION=ftp://46.21.173.143/domains/eengoedkopewebsite.be/public_html
export WERCKER_FTP_DEPLOY_USERNAME=webdesign
export WERCKER_FTP_DEPLOY_PASSWORD=WuPg6o2SzOatIVl5
export WERCKER_CACHE_DIR=~/Downloads

function success() { echo "$@";}
function debug() { echo "$@";}
function fail() { echo "$@";exit 1;}
export -f success debug fail
