Table of Contents
Introduction
Highlights
Features
Architecture
Getting Started
Option A: Run with Docker (recommended)
Option B: Run each service manually
Configuration
Troubleshooting
Contributing
License
Acknowledgements
Contact
Introduction

We are planning to build an app designed to assist people with creating and attending events, with the goal of improving the social life of all our app users by helping them connect with new people and spend time with shared interests and hobbies. Word of mouth is not a reliable and efficient way to plan and host events. Thus, we figured that an online application for doing so would help event hosts fill up their attendee lists and also for students to have a proper portal to browse and attend events that they are interested in. This app can be used to advertise the event hosts’ event, whether it is a professional event or a recreational event. We are planning to start with a web application, and then possibly change it to a mobile app if time permits.

Highlights

We decided this application was a good idea, first of all, because it caters to a specific need which is hosting events and spreading word about said events. That in itself makes it unique in our eyes, and worth attempting to create. We did not really come up with any alternatives, as we thought of this idea first. We also realized that there might be a considerable risk of meeting possibly dangerous individuals if it was available for anybody. Allowing university personnel acts as a safeguard against strangers. Some connections we build in university could help us throughout our entire lives, and students will need a better platform than existing forums to socialize and connect to each other. It also provides a convenient option for the type of event that people would like to attend eg. professional networking, socializing, etc. all under one umbrella. We organized the team initially using the Piazza post, and then created a Discord group chat to keep in touch with each other. In the Discord chat, we came up with all the specifics, such as the meeting times, sending each other important info like emails, etc. We meet 3 times a week.

Features
Event Creation and Management
Event Browsing and Searching
User Authentication and Authorization
Event Invitations and Sharing
Notifications and Reminders
Architecture

Rendezvous is made of three pieces that all need to be running at the same time:

Service	Folder	Tech	Default port
Frontend	rendezvous_app/	React	3000 (or 42070 via Docker)
Backend / API	backend_fastapi/	FastAPI (Python)	8000 (or 42069 via Docker)
Database	—	MySQL	3306

The frontend calls the FastAPI backend over HTTP (see rendezvous_app/src/Pages/fastapi.js), and the backend reads/writes users and events in MySQL via SQLAlchemy. Registering an account, logging in, hosting an event, adding friends, etc. all go through the backend — running npm start in rendezvous_app/ by itself only gets you the UI shell, with no working login or data.

Prerequisites

You need Docker Desktop installed and running. This project is set up to run as three coordinated containers (frontend, backend, MySQL) via Docker Compose, and that's the only setup path the team actively maintains. You do not need Node.js, Python, Poetry, or MySQL installed on your machine — Docker handles all of that inside the containers. (A manual, no-Docker setup is documented below for reference, but it's unsupported and more error-prone.)

macOS: brew install --cask docker
Windows/Linux: download the installer from the link above

After installing, open the Docker Desktop app once and wait for the whale icon in your menu bar / system tray to show it's running — the docker CLI won't work until the engine is up.

Getting Started
Option A: Run with Docker (recommended — this is how the app is meant to be run)

Docker builds and starts all three services together (MySQL, FastAPI, React) with one command, using the exact same versions and configuration for everyone — no need to install Python, Poetry, or MySQL yourself.

1. Install Docker Desktop (includes the Docker engine + Compose plugin):

sh
brew install --cask docker

Then open the Docker app from your Applications folder once, and wait until you see the whale icon in your menu bar (this means the background engine is running). You only need to do this once.

Note: brew install docker (no --cask) only installs the command-line client, not the engine — that's why docker-compose may say command not found. Also, current Docker versions use docker compose (a space, no hyphen) instead of the older standalone docker-compose binary.

2. Start everything from the repo root:

sh
git clone https://github.com/username/repo.git
cd Rendezvous-main
docker compose up --build

3. Open the app:

Frontend: http://localhost:42070
Backend API docs (Swagger UI): http://localhost:42069/docs

The database schema (accounts, events, etc.) is created automatically on first startup — no manual SQL needed.

To stop everything, press Ctrl+C, then run docker compose down (add -v to also wipe the database volume and start fresh).

Option B: Run each service manually (not recommended / unsupported)

Docker is the supported way to run this app — it avoids version mismatches across Python, Node, and MySQL, and it's the only setup that's actively tested. Use this option only if you specifically can't use Docker.

Prerequisites: Node.js + npm, Python 3.12+, Poetry, and a local MySQL server.

1. Database:

Start MySQL locally and create a database matching backend_fastapi/.env (MYSQL_DATABASE), or edit DATABASE_URL in that file to point at your own local MySQL user/database.

2. Backend:

sh
cd backend_fastapi
poetry install
poetry run uvicorn backend_fastapi.main:app --reload --port 8000

3. Frontend (in a new terminal):

sh
cd rendezvous_app
npm install
npm start

Make sure rendezvous_app/.env has REACT_APP_BACKEND_URL pointing at your backend (http://localhost:8000 if you used the command above).

Configuration

Environment variables live in .env files (already present for local dev, don't commit real secrets):

backend_fastapi/.env

MYSQL_DATABASE / MYSQL_USER / MYSQL_PASSWORD / MYSQL_ROOT_PASSWORD — MySQL credentials (used by the db container in Docker mode)
DATABASE_URL — full SQLAlchemy connection string the backend uses, e.g. mysql+pymysql://user:password@host/dbname
SECRET_KEY — used to sign login JWTs (generate your own with openssl rand -hex 32)

rendezvous_app/.env

REACT_APP_BACKEND_URL — URL the frontend uses to reach the backend
Troubleshooting
docker-compose: command not found — You have the CLI but not Docker Desktop/engine. Install Docker Desktop (see Prerequisites), open the Docker app, and use docker compose (space) instead of docker-compose (hyphen).
unknown flag: --build — This can happen if docker compose runs before the Docker engine has fully finished starting up right after installing Docker Desktop. Give it a minute (wait for the whale icon to settle) and try the command again.
Backend build fails with Error: The current project could not be installed: Readme path '/app/README.md' does not exist — Already fixed in this repo: backend_fastapi/pyproject.toml sets package-mode = false, which tells Poetry to just install dependencies instead of trying to package the backend itself (which was failing because the Dockerfile copies pyproject.toml/poetry.lock in before the rest of the source, including the README, is present). If you ever hit this again after editing pyproject.toml, check that package-mode = false is still set under [tool.poetry].
Frontend loads but login/register does nothing / network errors — The backend and/or database isn't running. Make sure docker compose up shows all three containers as healthy (mysql_db, fastapi_app, rendezvous_app), and check docker compose logs fastapi for errors.
Port already in use — Something else on your machine is using 3000, 3306, 8000, 42069, or 42070. Stop the conflicting process or change the port mapping in docker-compose.yml.
Made changes to backend or frontend code and don't see them — Try docker compose up --build again to force a rebuild, or docker compose down -v && docker compose up --build for a completely clean start (this also wipes the database).
Contributing

If you would like to contribute to this project, please follow the guidelines:

Fork the repository.
Create a new branch (git checkout -b feature-branch).
Commit your changes (git commit -m 'Add some feature').
Push to the branch (git push origin feature-branch).
Create a new Pull Request.

Please make sure your code adheres to the coding standards and includes appropriate tests

License

This project is licensed under the MIT License - see the LICENSE file for details.

Acknowledgments
The UTSC LAUNCH event team for inspiration
Our university staff and students who participated in the focus groups
Contact

If you have any questions, feel free to reach out to us through Github.
