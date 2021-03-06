all: package-lambda

help:
	@echo "Please use \`make <target>' where <target> is one of"
	@echo "  package-lambda      to package the lambda function"
	@echo "  create-lambda       to create the lambda function in AWS"
	@echo "  update-lambda       to update the lambda function in AWS. May be mismatched to the region deployed"
	@echo "  test-local          to test locally"
	@echo "  invoke-lambda       to invoke execution in AWS. May be mismatched to the region"


stats.zip: stats.py 
	# AWS Lambda NOW expects the script to be named as lambda_function.py
	cp stats.py lambda_function.py
	rm stats.zip
	rm -rf package/
	pip install --target ./package datadog
	cp -f citrixadcmetrics.py package/
	cd package && zip -r9 ../stats.zip .
	cd .. && zip -g stats.zip lambda_function.py
	rm -f lambda_function.py

package-lambda: stats.zip
	@echo "create/update lambda deployment package (stats.zip)"

update-lambda:  package-lambda
	@echo "update lambda deployment package"
	aws lambda update-function-code  --function-name ${LAMBDA_FUNCTION_NAME} --zip-file fileb://${PWD}/stats.zip

test-local:
	@echo "Testing locally"
	python-lambda-local -l lib/ -f lambda_handler -t 50 stats.py event.json

invoke-lambda:
	@echo "Invoking Lambda remote"
	aws lambda invoke --function-name ${LAMBDA_FUNCTION_NAME} --log-type Tail outfile.txt | grep LogResult | awk -F" " '{print $$2}' | sed 's/"//g' | sed 's/,//g'  | base64 --decode
