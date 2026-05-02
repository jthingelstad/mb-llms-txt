# {{ .Title | default (.Date.Format "2006-01-02") }}

*{{ .Date.Format "2006-01-02" }}{{ with .Params.author }} — {{ . }}{{ end }}* — {{ .Permalink }}

{{ .RawContent }}
{{ with .Params.tags }}
---
Tags: {{ delimit . ", " }}
{{ end }}
