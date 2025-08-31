# Claude Code Configuration

## Allowed Tools

```yaml
allowed_tools:
  - WebSearch
  - WebFetch:
      domain: developers.amadeus.com
  - Bash:
      patterns:
        - "chmod:*"
        - "pip install:*" 
        - "python:*"
        - "python3:*"
        - "./track_flights:*"
        - "./dump_flights:*"
        - "git init:*"
        - "git add:*"
        - "git commit:*"
        - "git push:*"
        - "git remote:*"
        - "git status:*"
        - "git log:*"
        - "git diff:*"
        - "gh repo create:*"
        - "gh auth:*"
        - "ls:*"
        - "cat:*"
        - "head:*"
        - "tail:*"
        - "grep:*"
        - "find:*"
        - "whoami:*"
        - "which:*"
        - "mv:*"
        - "cp:*"
        - "rm:*"
  - Read:
      patterns:
        - "/Users/nick/proj/flights/**"
        - "/tmp/flights-test/**" 
        - "/private/tmp/flights-test/**"
        - "/var/data/flights-*/**"
  - Write:
      patterns:
        - "/Users/nick/proj/flights/**"
        - "/tmp/flights-*"
  - Edit:
      patterns:
        - "/Users/nick/proj/flights/**"
  - MultiEdit:
      patterns:
        - "/Users/nick/proj/flights/**"
  - Glob:
      patterns:
        - "/Users/nick/proj/flights/**"
        - "/tmp/flights-*/**"
        - "/var/data/flights-*/**"
  - Grep:
      patterns:
        - "/Users/nick/proj/flights/**"
        - "/tmp/flights-*/**"
        - "/var/data/flights-*/**"
```

## Project Context

This is a flight price tracking project using the Amadeus API. The main components are:

- `track_flights`: Main script to fetch flight data and save to JSON
- `dump_flights`: Utility to read and analyze saved flight data  
- `config.yaml`: Flight route and search configuration
- `secrets-*.yaml`: API credentials (not tracked in git)

## Common Tasks

- Running flight searches: `./track_flights test /tmp/flights`
- Analyzing data: `./dump_flights /tmp/flights-test --sort-by price`
- Modifying configuration and scripts
- Git operations for version control
- Installing dependencies and testing functionality
