docker_build('conceptmasters/stict', '.',
    dockerfile='./Dockerfile.dev', ignore=[
        'trusted_roots',
        'tests',
        'mysql_data',
        'trillian/examples',
    ],
    live_update=[
        sync('.', '/app'),
        run('CGO_ENABLED=0 GOOS=linux go build -o /sti-ct ./cmd/stict'),
        restart_container(),
])
docker_compose('./docker-compose.yaml', profiles=["frontend"])