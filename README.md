##Разветрывание стенда Juice Shop
docker run -d --name juice-shop -p 3001:3000 bkimminich/juice-shop 
juice-shop

##Развертывание WebGoat
docker run -d --name webgoat -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 \
  -e TZ=Europe/Moscow webgoat/webgoat

##Клонирование исходников для SCA/SAST
cd targets
git clone --depth 1 https://github.com/WebGoat/WebGoat.git webgoat

##Установка Semgrep
brew install semgrep

##Сканирование semgrep в json(для анализа)
cd ~/saz-lab-Гусак/targets/juice-shop
semgrep scan \
  --config p/owasp-top-ten \
  --config p/security-audit \
  --config p/javascript \
  --json \
  -o ../../reports/sast/juice-shop.json \
  --metrics=off

##Сканирование в SARIF (для DefectDojo)
cd ~/saz-lab-Гусак/targets/juice-shop
semgrep scan \
  --config p/owasp-top-ten \
  --config p/security-audit \
  --config p/javascript \
  --sarif \
  -o ../../reports/sast/juice-shop.sarif \
  --metrics=off

##Сканирование WebGoat
cd ~/saz-lab-Гусак/targets/webgoat

# JSON
semgrep scan \
  --config p/owasp-top-ten \
  --config p/security-audit \
  --config p/java \
  --json \
  -o ../../reports/sast/webgoat.json \
  --metrics=off

# SARIF
semgrep scan \
  --config p/owasp-top-ten \
  --config p/security-audit \
  --config p/java \
  --sarif \
  -o ../../reports/sast/webgoat.sarif \
  --metrics=off

###Написание собственного правила Semgrep

##Создание директории правила
cd ~/saz-lab-Гусак
mkdir -p tools/semgrep-custom

##Создание файла правила

nano tools/semgrep-custom/rules.yaml

###Собственное правило на Juice Shop

cd ~/saz-lab-Гусак
semgrep scan \
  --config tools/semgrep-custom/rules.yaml \
  targets/juice-shop \
  --json \
  --output reports/sast/juice-shop-custom.json \
  --metrics=off

##Запуск на WebGoat
semgrep scan \
  --config tools/semgrep-custom/rules.yaml \
  targets/webgoat \
  --json \
  --output reports/sast/webgoat-custom.json \
  --metrics=off

## Запуск с выводом в терминал
semgrep scan --config tools/semgrep-custom/rules.yaml targets/juice-shop --metrics=off

##Анализ SARIF файла

# Структура SARIF
cat reports/sast/juice-shop.sarif | head -100

# Извлекаем все уязвимости с уровнем ERROR
cat reports/sast/juice-shop.json | jq '.results[] | select(.extra.severity=="ERROR") | {check_id, path, start: .start.line, severity: .extra.severity}'

##Заполняем triage_report.md
nano triage/triage_report.md

##Установка cdxgen
npm install -g @cyclonedx/cdxgen
cdxgen --version

## Генерация SBOM для Juice Shop
cd ~/saz-lab-Гусак/targets/juice-shop

cdxgen \
  -t javascript \
  -r \
  -o ../../reports/sca/juice-shop.cdx.json \
  --verbose

## Генерация SBOM для WebGoat
cd ~/saz-lab-Гусак/targets/webgoa
cdxgen \
  -t java \
  -r \
  -o ../../reports/sca/webgoat.cdx.json \
  --verbose

##Развёртывание Dependency-Track
cd ~/saz-lab-Гусак/tools/dependency-track
ls -la

##Запуск D-T
docker compose up -d

##Загрузка SBOM в Dependency-Track
curl -X POST "http://localhost:8081/api/v1/bom" \
  -H "X-Api-Key: $DT_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F "autoCreate=true" \
  -F "projectName=juice-shop" \
  -F "projectVersion=latest" \
  -F "bom=@reports/sca/juice-shop.cdx.json"
{"token":"42d591d6-f086-46ec-b31c-41c30b43eb36"}

curl -X POST "http://localhost:8081/api/v1/bom" \
  -H "X-Api-Key: $DT_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F "autoCreate=true" \
  -F "projectName=webgoat" \
  -F "projectVersion=latest" \
  -F "bom=@reports/sca/webgoat.cdx.json"
{"token":"57d22d30-f862-4e9c-8126-462caa57ff5f"}

##Установка Trivy
brew install trivy
trivy --version

##Сканирование Docker-образов (CA/BCA)
cd ~/saz-lab-Гусак

# Juice Shop
trivy image --format json --output reports/ca/juice-shop.image.json bkimminich/juice-shop

# WebGoat
trivy image --format json --output reports/ca/webgoat.image.json webgoat/webgoat

##Сканирование исходников (filesystem)
# Juice Shop
trivy fs --format json --output reports/ca/juice-shop.fs.json targets/juice-shop

# WebGoat
trivy fs --format json --output reports/ca/webgoat.fs.json targets/webgoat

##IaC сканирование
# Создать Dockerfile
mkdir -p targets/iac-demo

cat > targets/iac-demo/Dockerfile << 'EOF'
FROM ubuntu:18.04
USER root
ADD https://example.com/install.sh /tmp/install.sh
RUN chmod +x /tmp/install.sh && /tmp/install.sh
EXPOSE 22
ENV SECRET_KEY="hardcoded_secret_12345"
CMD ["/bin/bash"]
EOF

# Сканирование
trivy config --format json --output reports/iac/dockerfile.json targets/iac-demo

## Генерация SBOM через Trivy
trivy image --format cyclonedx --output reports/sca/juice-shop.trivy.cdx.json bkimminich/juice-shop

## Baseline scan
docker run --rm --network host \
  -v $(pwd)/reports/dast:/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
    -t http://localhost:3001 \
    -J juice-shop.baseline.json \
    -r juice-shop.baseline.html \
    -x juice-shop.baseline.xml

##  Full scan
docker run --rm --network host \
  -v $(pwd)/reports/dast:/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-full-scan.py \
    -t http://localhost:3001 \
    -J juice-shop.full.json \
    -r juice-shop.full.html \
    -x juice-shop.full.xml \
    -j -m 5 

##Automation Framework с аутентификацией
cd ~/saz-lab-Гусак

docker run --rm --network host \
  -v $(pwd)/tools/zap:/zap/plans:ro \
  -v $(pwd)/reports/dast:/zap/wrk:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap.sh -cmd -autorun /zap/plans/juice-shop-auth.yaml
  
  
