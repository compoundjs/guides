## Deploy

Deployment process should match the following requirements checklist:

- node process is running as daemon, restarted after successful code deploy
- some files (images, logs, configs) shared across releases
- dependencies updated
- easy to rollback broken deploy (with all dependencies)
- logs easily accesible
- errors logged to separate server
- process should be started on server startup


