# VDX Proxy Konfiguration - Forklaring

Dette dokument beskriver konfigurationen i `config.json` filen og port-definitionerne i `docker-compose.yml`.

## Port-mapping oversigt

| Service       | Container Port | Host Port | Beskrivelse              |
| ------------- | -------------- | --------- | ------------------------ |
| VideoAPI      | 8080           | 8095      | Håndtering af videomøder |
| SmsAPI        | 8081           | 8096      | SMS-notifikationer       |
| DeviceReg API | 8082           | 8082      | Enheds-administration    |
| History API   | 8083           | 8083      | Historiske møde-data     |

## Forklaring af config.json

### Generel konfiguration

- `logging.logs.default.level`: Logningsniveau for Kitcaddy
- `admin.disabled`: Deaktiverer admin-interfacet

### VideoAPI konfiguration (`video-api`)

- `listen`: Intern container port til VideoAPI (8080)
- `match.path`: Fanger alle stier under denne rute
- `handler`: OIO IDWS REST Web Service Consumer
- `mongo_host`: Henviser til MongoDB service i docker-compose
- `mongo_db`: Database navn til token caching
- `sts_url`: URL til Security Token Service
- `client_cert_file`: Klientcertifikat til mTLS
- `client_key_file`: Klientnøgle til mTLS
- `trust_cert_files`: STS server-certifikat til verifikation
- `service_endpoint`: Videoapi endpoint
- `service_audience`: Audience parameter til security token
- `reverse_proxy`: Viderestiller requests til backend
- `upstreams.dial`: Target server og port
- `headers.request.set.Host`: Sætter Host header for requests

### SmsAPI konfiguration (`sms-api`)

- `listen`: Intern container port til SmsAPI (8081)
- `mongo_db`: Database til token caching for sms-api
- `service_endpoint`: SMS endpoint
- `service_audience`: Samme audience som VideoAPI
- `insecure_skip_verify`: OBS: Deaktiverer certifikat verifikation
- `upstreams.dial`: Samme backend som VideoAPI

### DeviceReg API konfiguration (`device-api`)

- `listen`: Intern container port til DeviceReg API (8082)
- `mongo_db`: Database til token caching for device-api
- `service_endpoint`: DeviceReg endpoint
- `service_audience`: Samme audience som VideoAPI
- `upstreams.dial`: Samme backend som VideoAPI

### History API konfiguration (`history-api`)

- `listen`: Intern container port til History API (8083)
- `mongo_db`: Database til token caching for history-api
- `service_endpoint`: History API endpoint
- `service_audience`: Samme audience som VideoAPI
- `upstreams.dial`: History API benytter et andet backend (historyapi.vconf-stage.dk)

## Ændring fra Stage til Produktion

For at skifte fra stage- til produktionsmiljø, skal følgende URL'er ændres:

1. `sts_url`:

   - Stage: `https://sts.vconf-stage.dk/sts/service/sts`
   - Prod: `https://sts.vconf.dk/sts/service/sts`

2. `service_endpoint`:

   - VideoAPI Stage: `https://videoapi.vconf-stage.dk/videoapi`
   - VideoAPI Prod: `https://videoapi.vconf.dk/videoapi`
   - SmsAPI Stage: `https://videoapi.vconf-stage.dk/sms`
   - SmsAPI Prod: `https://videoapi.vconf.dk/sms`
   - DeviceReg Stage: `https://videoapi.vconf-stage.dk/devicereg`
   - DeviceReg Prod: `https://videoapi.vconf.dk/devicereg` (Bemærk: Aktuelt ikke aktiv)
   - History Stage: `https://historyapi.vconf-stage.dk/`
   - History Prod: `https://historyapi.vconf.dk/`

3. `upstreams.dial` og tilhørende `Host` header:
   - Ændr alle forekomster af `-stage` i domænenavnene
