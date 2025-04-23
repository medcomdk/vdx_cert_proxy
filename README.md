# VDX Certifikat Proxy Service

## Formål og fordele

Denne service fungerer som en proxy, der gør det nemt at integrere din applikation med MedCom's VDX API'er (VideoAPI, SmsAPI, DeviceReg API og History API). Servicen:

- **Forenkler sikkerhed**: Håndterer automatisk certifikatudveksling mellem VDX og din applikation
- **Skjuler kompleksitet**: Din applikation behøver ikke at kende til sikkerhedslaget mellem systemerne
- **Nem opsætning**: Du skal blot tilføje dit certifikat og privatnøgle for at komme i gang
- **Lokal udvikling**: Tillader udvikling og test mod VDX API'er fra dit lokale miljø

Teknisk set er løsningen baseret på Kitcaddy WSC-proxy:

- [GitHub repository](https://github.com/KvalitetsIT/kitcaddy)
- [Docker Hub](https://hub.docker.com/r/kvalitetsit/kitcaddy)

## Hurtig start guide

### 1. Forudsætninger

- Docker og Docker Compose installeret
- Et gyldigt klientcertifikat konfigureret i VDX STS ([vejledning her](https://medcom.dk/systemforvaltning/videoknudepunktet-vdx/vdx-vejledninger/certifikatadgang-til-vdx-api/))

### 2. Certifikat og nøgle opsætning

1. Placer din privatnøgle som `client.key` i `config`-mappen
2. Placer dit klientcertifikat som `client.crt` i `config`-mappen
3. **Vigtigt**: Filen `/trust/sts.cer` er en statisk fil der er nødvendig for at tjenesten fungerer (rør ikke ved denne)

### 3. Start tjenesten

Kør servicen fra roden af projektmappen:

```shell
docker compose up
```

Proxy'en vil nu lytte på følgende lokale porte:

- Port 8080: VideoAPI
- Port 8081: SmsAPI
- Port 8082: DeviceReg API
- Port 8083: History API

## API-information og miljøer

### VDX tilgængelige API'er

Her er en oversigt over de API'er, du kan få adgang til via denne proxy:

| API                                                                     | Beskrivelse              | Stage Port |
| ----------------------------------------------------------------------- | ------------------------ | ---------- |
| [VideoAPI](https://api-docs.vconf-stage.dk/videoapi/v1/videoapi/)       | Håndtering af videomøder | 8080       |
| [SmsAPI](https://api-docs.vconf-stage.dk/videoapi/v1/sms/)              | SMS-notifikationer       | 8081       |
| [DeviceReg API](https://api-docs.vconf-stage.dk/videoapi/v1/devicereg/) | Enheds-administration    | 8082       |
| [History API](https://api-docs.vconf-stage.dk/videoapi/v1/statistics/)  | Historiske møde-data     | 8083       |

### Skift mellem Stage- og Produktionsmiljø

For at skifte fra stage-miljø til produktionsmiljø:

1. Rediger `config/config.json` filen
2. Erstat alle forekomster af "-stage" i URL'erne

Eksempler:

- `https://sts.vconf-stage.dk/sts/service/sts` → `https://sts.vconf.dk/sts/service/sts`
- `https://videoapi.vconf-stage.dk/videoapi` → `https://videoapi.vconf.dk/videoapi`

**Bemærk**: DeviceReg API er aktuelt ikke aktivt i produktionsmiljøet.

### API-dokumentation

#### Stage-miljø

- Video API: [https://api-docs.vconf-stage.dk/videoapi/v1/videoapi/](https://api-docs.vconf-stage.dk/videoapi/v1/videoapi/)
- SMS API: [https://api-docs.vconf-stage.dk/videoapi/v1/sms/](https://api-docs.vconf-stage.dk/videoapi/v1/sms/)
- DeviceReg API: [https://api-docs.vconf-stage.dk/videoapi/v1/devicereg/](https://api-docs.vconf-stage.dk/videoapi/v1/devicereg/)
- History API: [https://api-docs.vconf-stage.dk/videoapi/v1/statistics/](https://api-docs.vconf-stage.dk/videoapi/v1/statistics/)

#### Produktionsmiljø

- Video API: [https://api-docs.vconf.dk/videoapi/v1/videoapi/](https://api-docs.vconf.dk/videoapi/v1/videoapi/)
- SMS API: [https://api-docs.vconf.dk/videoapi/v1/sms/](https://api-docs.vconf.dk/videoapi/v1/sms/)
- DeviceReg API: [https://api-docs.vconf.dk/videoapi/v1/devicereg/](https://api-docs.vconf.dk/videoapi/v1/devicereg/) (Aktuelt ikke aktiv)
- History API: [https://api-docs.vconf.dk/videoapi/v1/statistics/](https://api-docs.vconf.dk/videoapi/v1/statistics/)

## Kald af API endpoints (eksempler)

Alle nedenstående eksempler antager, at Docker-containeren kører på din maskine. Eksemplerne bruger `curl` kommandoer, men du kan også bruge andre HTTP-klienter.

### VideoAPI

#### Opret et møde

```shell
# Husk at redigere requests/create_meeting.json med korrekt e-mail
curl -v -X POST -H "Content-Type: application/json" -d @requests/create_meeting.json http://localhost:8080/videoapi/meetings
```

#### Hent mødedetaljer

```shell
curl -v http://localhost:8080/videoapi/meetings/{meeting_uuid}
```

### SmsAPI

#### Send SMS med mødelink

```shell
# Husk at redigere requests/send_sms.json med korrekt telefonnummer
curl -v -X POST -H "Content-Type: application/json" -d @requests/send_sms.json -H "accept: application/json" http://localhost:8081/sms/v1/meeting/{meeting_uuid}
```

#### Hent SMS-status

```shell
curl -v -H "accept: application/json" http://localhost:8081/sms/v1/meeting/{meeting_uuid}
```

### DeviceReg API

#### Opret en enhed

```shell
# Husk at redigere requests/create_device.json med korrekte enhedsoplysninger
curl -v -X POST -H "Content-Type: application/json" -d @requests/create_device.json -H "accept: application/json" http://localhost:8082/v1/device
```

#### Hent enhedsinformation

```shell
curl -v -H "accept: application/json" http://localhost:8082/v1/device/{device_id}
```

#### Liste alle enheder

```shell
curl -v -H "accept: application/json" http://localhost:8082/v1/device
```

### History API

**Bemærk:** History API kræver en API-nøgle som header. Denne nøgle kan findes i VDX Management webappen under organisationsafsnittet.

#### Hent mødehistorik

```shell
curl -v -H "accept: application/json" -H "X-API-KEY: {din_api_nøgle}" "http://localhost:8083/apikey/history/v1/conference?start_time__gte=2023-01-01T00:00:00Z&end_time__lt=2023-12-31T23:59:59Z&limit=100"
```

## Fejlsøgning

### Almindelige problemer

1. **Certifikatfejl**: Kontroller at dit certifikat og privatnøgle er korrekt placeret og navngivet
2. **Connectivity-problemer**: Kontroller at du kan nå VDX-serverne fra dit netværk
3. **Docker-problemer**: Sørg for at Docker-containeren kører med `docker ps`

### Logs

For at se logs fra WSC-proxyen:

```shell
docker compose logs -f wsc
```
