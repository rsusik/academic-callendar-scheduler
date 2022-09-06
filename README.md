# Academic calendar scheduler

## About

Academic calendar scheduler was created for IdeaBox initiative that is 
annual event held at Lodz University of Technology (TUL) 
where employees and students can submit their ideas to improve 
working and studying at the university.

Available at: (TODO)

The algorithm which is a core of the project is base of research paper.

## Usage
* Open the scheduler in web browser
* Provide semester start date
* Provide how many weeks the term has and how many of them are in the first part
* Add/remove days to/from free days (holidays) by clicking the days on calendar
* There is an option to switch between two languages (English and Polish)

## Requirements (for development)

* JavaScript
* Node (v16.14.0)
* Npm (v8.15.1)
* Quasar (v1.2.1)

_The code was tested on Linux 64-bit OS (Fedora and Arch distributions) and should also work on other systems (such as Windows and Mac OS) if all requirements are met, but it was not verified._


## Running (for development)

### Install the dependencies

```bash
npm install
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)

```bash
quasar dev
```

### Lint the files

```bash
npm run lint
```

### Format the files

```bash
npm run format
```

### Build the app for production

```bash
quasar build
```
