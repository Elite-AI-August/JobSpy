# JobSpy AIO Scraper

## Features

- Scrapes job postings from **LinkedIn**, **Indeed** & **ZipRecruiter** simultaneously
- Returns jobs as JSON or CSV with title, location, company, description & other data
- Imports directly into **Google Sheets**
- Optional JWT authorization

![jobspy_gsheet](https://github.com/cullenwatson/JobSpy/assets/78247585/9f0a997c-4e33-4167-b04e-31ab1f606edb)

### API

POST `/api/v1/jobs/`
### Request Schema
```plaintext
Required
├── site_type (List[enum]): linkedin, zip_recruiter, indeed
└── search_term (str)
Optional
├── location (int)
├── distance (int)
├── job_type (enum): fulltime, parttime, internship, contract
├── is_remote (bool)
├── results_wanted (int): per site_type
├── easy_apply (bool): only for linkedin
└── output_format (enum): json, csv, gsheet
```
### Request Example
```json
"site_type": ["indeed", "linkedin"],
"search_term": "software engineer",
"location": "austin, tx",
"distance": 10,
"job_type": "fulltime",
"results_wanted": 15
"output_format": "gsheet"
```
### Response Schema
```plaintext
site_type (enum): 
JobResponse
├── success (bool)
├── error (str)
├── jobs (List[JobPost])
│   └── JobPost
│       ├── title (str)
│       ├── company_name (str)
│       ├── job_url (str)
│       ├── location (object)
│       │   ├── country (str)
│       │   ├── city (str)
│       │   ├── state (str)
│       ├── description (str)
│       ├── job_type (enum)
│       ├── compensation (object)
│       │   ├── interval (CompensationInterval): yearly, monthly, weekly, daily, hourly
│       │   ├── min_amount (float)
│       │   ├── max_amount (float)
│       │   └── currency (str)
│       └── date_posted (datetime)
│
├── total_results (int)
└── returned_results (int) 
```
### Response Example (GOOGLE SHEETS)
```json
{
    "status": "Successfully uploaded to Google Sheets",
    "error": null,
    "linkedin": null,
    "indeed": null,
    "zip_recruiter": null
}
```
### Response Example (JSON)
```json
{
    "indeed": {
        "success": true,
        "error": null,
        "jobs": [
            {
                "title": "Software Engineer",
                "company_name": "INTEL",
                "job_url": "https://www.indeed.com/jobs/viewjob?jk=a2cfbb98d2002228",
                "location": {
                    "country": "USA",
                    "city": "Austin",
                    "state": "TX",
                },
                "description": "Job Description Designs, develops, tests, and debugs..."
                "job_type": "fulltime",
                "compensation": {
                    "interval": "yearly",
                    "min_amount": 209760.0,
                    "max_amount": 139480.0,
                    "currency": "USD"
                },
                "date_posted": "2023-08-18T00:00:00"
            }, ...
        ],
        "total_results": 845,
        "returned_results": 15
    },
    "linkedin": {
        "success": true,
        "error": null,
        "jobs": [
            {
                "title": "Software Engineer 1",
                "company_name": "Public Partnerships | PPL",
                "job_url": "https://www.linkedin.com/jobs/view/3690013792",
                "location": {
                    "country": "USA",
                    "city": "Austin",
                    "state": "TX",
                },
                "description": "Public Partnerships LLC supports individuals with disabilities..."
                "job_type": null,
                "compensation": null,
                "date_posted": "2023-07-31T00:00:00"
            }, ...
        ],
        "total_results": 2000,
        "returned_results": 15
    }
}
```
### Response Example (CSV)
```
Site, Title, Company Name, Job URL, Country, City, State, Job Type, Compensation Interval, Min Amount, Max Amount, Currency, Date Posted, Description
indeed, Software Engineer, INTEL, https://www.indeed.com/jobs/viewjob?jk=a2cfbb98d2002228, USA, Austin, TX, fulltime, yearly, 209760.0, 139480.0, USD, 2023-08-18T00:00:00, Job Description Designs...
linkedin, Software Engineer 1, Public Partnerships | PPL, https://www.linkedin.com/jobs/view/3690013792, USA, Austin, TX, , , , , , 2023-07-31T00:00:00, Public Partnerships LLC supports...
```

## Installation
### Docker Setup
_Requires [Docker Desktop](https://www.docker.com/products/docker-desktop/)_

[JobSpy API Image](https://ghcr.io/cullenwatson/jobspy:latest) is continuously updated and available on GitHub Container Registry.
  
To pull the Docker image:

```bash
docker pull ghcr.io/cullenwatson/jobspy:latest
```
  
#### Params
  
By default:
* Port: `8000`
* Google sheet name: `JobSpy`
* Uses relative path of `client_secret.json` (for Google Sheets, see below to obtain)

  
To run the image with these default settings, use:
    
Example (Windows):
```bash
docker run -v %cd%/client_secret.json:/app/client_secret.json -p 8000:8000 ghcr.io/cullenwatson/jobspy
```
  
Example (Unix):
```bash
docker run -v $(pwd)/client_secret.json:/app/client_secret.json -p 8000:8000 ghcr.io/cullenwatson/jobspy
```
  
#### Using custom params

  Example: 
   * Port: `8030`
   * Google sheet name: `CustomName`
   * Absolute path of `client_secret.json`: `C:\config\client_secret.json`

  To pass these custom params:
```bash
docker run -v C:\config\client_secret.json:/app/client_secret.json -e GSHEET_NAME=CustomName -e PORT=8030 -p 8030:8030 ghcr.io/cullenwatson/jobspy
```
  
### Python installation (alternative to Docker)
_Python version >= [3.10](https://www.python.org/downloads/release/python-3100/) required_  
1. Clone this repository `git clone https://github.com/cullenwatson/jobspy`
2. Install the dependencies with `pip install -r requirements.txt`
4. Run the server with `uvicorn main:app --reload`
  
### Google Sheets Setup
  
#### Obtaining an Access Key: [Video Guide](https://youtu.be/w533wJuilao?si=5u3m50pRtdhqkg9Z&t=43)
  * Enable the [Google Sheets & Google Drive API](https://console.cloud.google.com/)
  * Create credentials -> service account -> create & continue
  * Select role -> basic: editor -> done
  * Click on the email you just created in the service account list
  * Go to the Keys tab -> add key -> create new key -> JSON -> Create
  
#### Using the key in the repo
  * Copy the key file into the JobSpy repo as `client_secret.json`
  * Go to [my template sheet](https://docs.google.com/spreadsheets/d/1mOgb-ZGZy_YIhnW9OCqIVvkFwiKFvhMBjNcbakW7BLo/edit?usp=sharing): File -> Make a Copy -> Rename to JobSpy
  * Share the Google sheet with the email located in the field `client_email` in the `client_secret.json` above with editor rights
  * If you changed the name of the sheet:
    - Python install: add `.env` in the repo and add `GSHEET_NAME` param with the sheet name as the value, e.g. `GSHEET_NAME=CustomName`
    - Docker install: use custom param `-e GSHEET_NAME=CustomName` in `docker run` (see above)
  
### How to call the API
  
#### [Postman](https://www.postman.com/downloads/) (preferred):
To use Postman:
1. Locate the files in the `/postman/` directory.
2. Import the Postman collection and environment JSON files.
  
#### Swagger UI:
Or you can call the API with the interactive documentation at [localhost:8000/docs](http://localhost:8000/docs).
  
## FAQ
  
### I'm having issues with my queries. What should I do?
  
Try reducing the number of `results_wanted` and/or broadening the filters. If issues still persist, feel free to submit an issue. 
  
### I'm getting response code 429. What should I do?
You have been blocked by the job board site for sending too many requests. Wait a couple seconds or use a VPN.
  
### How to enable auth?
  
Change `AUTH_REQUIRED` in `/settings.py` to `True`
  
The auth uses [supabase](https://supabase.com). Create a project with a `users` table and disable RLS.  
  
<img src="https://github.com/cullenwatson/jobspy/assets/78247585/03af18e1-5386-49ad-a2cf-d34232d9d747" width="500">
  
Add these three environment variables:
  
- `SUPABASE_URL`: go to project settings -> API -> Project URL  
- `SUPABASE_KEY`: go to project settings -> API -> service_role secret
- `JWT_SECRET_KEY` - type `openssl rand -hex 32` in terminal to create a 32 byte secret key
  
Use these endpoints to register and get an access token: 
  
![image](https://github.com/cullenwatson/jobspy/assets/78247585/c84c33ec-1fe8-4152-9c8c-6c4334aecfc3)

