# OpenCode

github  :https://github.com/anomalyco/opencode
web: https://opencode.ai/

## Installation

```
curl -fsSL https://opencode.ai/install | bash
```

### The real installation location

```
/home/devuser/.opencode/bin/opencode
```


### Start opencode

**Attention:   cd  <project folder> first**

```bash
cd <project folder>
opencode
```


## Starting

```
opencode models
```


```
opencode auth login
```

```
vi $HOME/.config/opencode/opencode.json 
```

```
$ more  $HOME/.config/opencode/opencode.json 
{
  "$schema": "https://opencode.ai/config.json",
  "model": "google/gemini-2.5-flash",
  "autoupdate": true
}
```