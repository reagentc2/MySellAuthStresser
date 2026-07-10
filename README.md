# bypass

simple cloudflare challenge solver + http load tester

uses playwright (real firefox) to solve cf challenges, grabs the cookies, then hands off to a go binary for the actual load generation part. go is way faster at raw rps than node so it made sense to split it up.

## how it works

1. **node.js** launches a bunch of headless firefox instances in parallel
2. each one navigates to the target, waits for cloudflare to do its thing, grabs the cookies
3. cookies + user agents get dumped to `sessions.json`
4. **go binary** picks up from there and does the actual http blasting with goroutines
5. results get printed at the end

if the go binary isnt built it falls back to a node.js dispatcher automatically.

## requirements

- node.js 18+
- playwright (`npm install`)
- go 1.21+ (only needed to build the loadgen binary)

## install

```bash
git clone https://github.com/yourname/bypass.git
cd bypass
npm install
cd loadgen
go build -o loadgen.exe main.go
cd ..
```

## usage

basic run with defaults (30 sessions, 1000 rps, 180s):
```bash
node bypass.js
```

custom params:
```bash
node bypass.js --target https://example.com --sessions 50 --rps 2000 --duration 300
```

skip go and use node.js dispatcher only:
```bash
node bypass.js --go false
```

hit multiple paths:
```bash
node bypass.js --paths /,/,/api,/login
```

## flags

| flag | default | description |
|------|---------|-------------|
| `--target` | `https://mysellauth.com/` | url to test against |
| `--sessions` | `30` | number of parallel browser sessions |
| `--rps` | `1000` | target requests per second |
| `--duration` | `180` | how long to run (seconds) |
| `--concur` | `8` | sockets per session |
| `--paths` | `/` | comma-separated paths to hit |
| `--method` | `GET` | http method |
| `--go` | `true` | use go load generator (set to `false` to disable) |
| `--out` | `loadtest-stats.json` | output json file path |

## output files

- `sessions.json` — cookies + user agents passed from node to go
- `loadtest-stats.json` — final stats report
- `loadtest-timeline.csv` — per-second metrics timeline

## project layout

```
bypass/
├── bypass.js          # challenge solver + dispatcher
├── package.json
├── loadgen/
│   ├── main.go        # go load generator
│   ├── go.mod
│   └── loadgen.exe    # built binary
└── README.md
```

## notes

- each session gets a random user agent from a pool of 20 (firefox, chrome, edge, opera, safari across windows/mac/linux/ios)
- sessions that get 503/429/errors get killed and replaced automatically
- the go dispatcher uses a ticker for rate limiting instead of just spamming as fast as possible
- round-robin session selection so one session doesnt get hammered

## license

mit
