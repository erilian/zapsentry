# Sentry client for zap logger

Integration of sentry client into zap.Logger is pretty simple:
```golang
func modifyToSentryLogger(log *zap.Logger, DSN string) *zap.Logger {
	cfg := zapsentry.Configuration{
		Level: zapcore.ErrorLevel, //when to send message to sentry
		EnableBreadcrumbs: true, // enable sending breadcrumbs to Sentry
		BreadcrumbLevel: zapcore.InfoLevel, // at what level should we sent breadcrumbs to sentry
		Tags: map[string]string{
			"component": "system",
		},
		DynamicTags: []string{ // list of fields to transfer in event tags from extras
			"traceID",
			"entityName",
		},
	}
	core, err := zapsentry.NewCore(cfg, zapsentry.NewSentryClientFromDSN(DSN))

	// to use breadcrumbs feature - create new scope explicitly
	log = log.With(zapsentry.NewScope())

	//in case of err it will return noop core. so we can safely attach it
	if err != nil {
		log.Warn("failed to init zap", zap.Error(err))
	}
	return zapsentry.AttachCoreToLogger(core, log)
}
```
