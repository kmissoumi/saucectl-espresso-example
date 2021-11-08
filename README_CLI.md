
```sh
# step 1
# clone && cd
# git clone https://github.com/kmissoumi/saucectl-espresso-example
git clone git@github.com:kmissoumi/saucectl-espresso-example.git
cd saucectl-espresso-example  

  
# install and update as needed
# curl -L https://saucelabs.github.io/saucectl/install | bash
# brew install saucectl
#
# release 0.73.1
saucectl --version  


# set credentials to environment variables
SAUCE_USERNAME=grogu
SAUCE_ACCESS_KEY=  

# run test
# this will upload the demo and test apps to your account
saucectl run --verbose && echo 'Go to level 2!' || echo 'STOP and send logs!'  


# level 2
# collect the uploaded app ids and set to variable
# trust me, this is the way
protoHost="https://api.us-west-1.saucelabs.com"
userAuth="${SAUCE_USERNAME}:${SAUCE_ACCESS_KEY}"

# call the sauce storage api
# ask jq to clean up mickey mouse response
storageApiResponse=$(curl --silent --user ${userAuth} \
  --request GET "${protoHost}/v1/storage/files?kind=android" \
  | jq  '.items[]|del(.metadata.icon)')

# we did it!..kinda maybe almost!
appId=$(jq -r 'select(.name == "calc.apk")|.id' <<< ${storageApiResponse})
testAppId=$(jq -r 'select(.name == "calc-success.apk")|.id' <<< ${storageApiResponse})

# we still need to have some configuration in yaml!? (╯°□°）╯︵ ┻━┻
echo "apiVersion: v1alpha\nregion: us-west-1" > .sauce/fig.yml  


# step 3 (finally!)
# this is the cli version of the test we ran in step 1
# instead of attempting to re-upload the app, we reference the file id
saucectl run espresso \
  --config .sauce/fig.yml \
  --ccy 10 \
  --app storage:${appId}\
  --testApp storage:${testAppId} \
  --device name="Google Pixel.*",deviceType=PHONE,private=false \
  --artifacts.download.when always \
  --artifacts.download.match junit.xml \
  --artifacts.download.directory "./artifacts" \
  --name "007 Espresso Support" \
  --build "moonraker" \
  --tags "e2e,draxCorp,theOtherJaws" \
  --verbose \
  --testOptions.class com.example.android.testing.androidjunitrunnersample.CalculatorAddParameterizedTest \
  --testOptions.class com.example.android.testing.androidjunitrunnersample.CalculatorInstrumentationTest  


# everything in its right place? let us clean up
# http response status codes will be written to standard out
curl --request DELETE --silent --user ${userAuth} \
  --output /dev/null --write-out '%{http_code}\n' "${protoHost}/v1/storage/files/${appId}" \
  --output /dev/null --write-out '%{http_code}\n' "${protoHost}/v1/storage/files/${testAppId}"
```
