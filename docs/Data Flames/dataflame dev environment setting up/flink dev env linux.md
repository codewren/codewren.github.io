# Flink Dev Env (Linux)

#### Dev Env Directory Layout <a href="#dev-env-directory-layout" id="dev-env-directory-layout"></a>

Copy

```
// Directory Layout
devws
├── devenv
│   └── flinkdev
├── launch
│   └── flink-1.19.1
├── opt
│   └── jdk-11.0.23+9
└── packages
```

#### JDK 11 <a href="#jdk-11" id="jdk-11"></a>

[![Logo](https://codewren.gitbook.io/dataflame/~gitbook/image?url=https%3A%2F%2Fadoptium.net%2Ficons%2Ficon-512x512.png%3Fv%3D3c8725a99800951594204e508d9aff1e\&width=20\&dpr=4\&quality=100\&sign=c10edea9\&sv=2)Latest Releases | Adoptiumadoptium](https://adoptium.net/temurin/releases/?version=11)

#### Flink <a href="#flink" id="flink"></a>

Flink 1.19

[![Logo](https://codewren.gitbook.io/dataflame/~gitbook/image?url=https%3A%2F%2Fflink.apache.org%2Ffavicon.png\&width=20\&dpr=4\&quality=100\&sign=200e7bec\&sv=2)Downloads](https://flink.apache.org/downloads/)

**Maven 3.8.8**

[![Logo](https://codewren.gitbook.io/dataflame/~gitbook/image?url=https%3A%2F%2Fmaven.apache.org%2Ffavicon.ico\&width=20\&dpr=4\&quality=100\&sign=7d1b26b\&sv=2)Maven – Download Apache Maven](https://maven.apache.org/download.cgi)

#### Env setup scripts <a href="#env-setup-scripts" id="env-setup-scripts"></a>

Copy

```
// Flink Dev env
#!/bin/bash

# Function to find shallowest devws directory using BFS
find_shallowest_devws() {
  # Find all directories under the current directory
  local dir="$1"
  local all_dirs=$(find $dir -maxdepth 3 -type d )

  # Sort directories by depth (shallowest first)
  sorted_dirs=$(echo "$all_dirs" | sort -n -k 1)


  ending_string="/devws"

  # Get the length of the ending string
  ending_length=${#ending_string}

  # Process directories in the sorted order (simulates BFS)
  for entry in $sorted_dirs; do
     # You can process the directory here
     #echo "Processing: $entry"
     # Check if the ending string matches the last characters of the variable string
     if [[ ${entry:(-ending_length)} == $ending_string ]]; then
        echo $entry
        break
     fi
  done

}

# Function to set application env variable
app_env_set() {
  base_dir="$1"


  # JAVA_HOME and PATH setting
  JAVA_HOME=${base_dir}/opt/jdk-11.0.23+9
  export JAVA_HOME

  #PATH
  export PATH=$JAVA_HOME/bin:$PATH

  ###
  ### MAVEN 
  MVN_HOME=${base_dir}/opt/apache-maven-3.8.8
  export PATH=$MVN_HOME/bin:$PATH
  

  #CMD prompt
  export PS1="(JDK11)\e[32m\u@\h \e[0m:\w\$ "  # Green username and hostname, normal text for directory and $

  # show key informaiton
  echo Current Path :  $PATH
  echo Java Version .......
  java -version
  echo MVN Version .......
  mvn --version

}

dir=$HOME

# Check if directory exists
if [[ ! -d "$dir" ]]; then
  echo "Error: Directory '$dir' does not exist."
  exit 1
fi

# Find shallowest devws directory
shallowest_devws=$(find_shallowest_devws "$dir")

# check the result (if found)
if [[ -n "$shallowest_devws" ]]; then
  echo "devws directory: $shallowest_devws"
  app_env_set $shallowest_devws 

else
  echo "No devws directory found in '$dir'."
fi

#end
```

**Flink Single Node Cluster Configuration**

Copy

```
flink-1.19.1/conf
168 
169 rest:
170   # The address to which the REST client will connect to
171   address: localhost
172   # The address that the REST & web server binds to
173   # By default, this is localhost, which prevents the REST & web server from
174   # being able to communicate outside of the machine/container it is running on.
175   #
176   # To enable this, set the bind address to one that has access to outside-facing
177   # network interface, such as 0.0.0.0.
178   #bind-address: localhost
179   bind-address: 0.0.0.0
180   # # The port to which the REST client connects to. If rest.bind-port has
181   # # not been specified, then the server will bind to this port as well.
182   # port: 8081
183   # # Port range for the REST and web server to bind to.
184   # bind-port: 8080-8090
185 
186 # web:
187 #   submit:
188 #     # Flag to specify whether job submission is enabled from the web-based
189 #     # runtime monitor. Uncomment to disable.
190 #     enable: false
191 #   cancel:
192 #     # Flag to specify whether job cancellation is enabled from the web-based
193 #     # runtime monitor. Uncomment to disable.
194 #     enable: false
195 
196 #==============================================================================
```

[\
](https://codewren.gitbook.io/dataflame/dataflame-dev-environment-setting-up/readme)
