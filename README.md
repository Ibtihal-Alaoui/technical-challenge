# technical-challenge
<h2 class="code-line" data-line-start=1 data-line-end=2 ><a id="Requirements_1"></a>Requirements</h2>
<ul>
<li class="has-line-data" data-line-start="3" data-line-end="4">Docker installed and running</li>
<li class="has-line-data" data-line-start="4" data-line-end="5">Kind installed and running</li>
<li class="has-line-data" data-line-start="5" data-line-end="6">kubectl installed</li>
</ul>
<h2 class="code-line" data-line-start=8 data-line-end=9 ><a id="User_Story_1_8"></a>User Story 1</h2>
<h3 class="code-line" data-line-start=9 data-line-end=10 ><a id="Using_the_default_configuration_9"></a>Using the default configuration</h3>
<ul>
<li class="has-line-data" data-line-start="10" data-line-end="11">Create a Kubernetes Secret to secure login credentials</li>
</ul>
<pre><code class="has-line-data" data-line-start="12" data-line-end="14" class="language-sh">kubectl create secret generic bonita-credentials --from-literal=TENANT_LOGIN=&lt;your-secured-tenant-login&gt; --from-literal=TENANT_PASSWORD=&lt;your-secured-tenant-password&gt;
</code></pre>
<p class="has-line-data" data-line-start="14" data-line-end="15">This command creates a Secret named bonita-credentials with two key-value pairs for TENANT_LOGIN and TENANT_PASSWORD.</p>
<ul>
<li class="has-line-data" data-line-start="16" data-line-end="17">Create a file named bonita-deployment1.yaml and add the following content</li>
</ul>
<pre><code class="has-line-data" data-line-start="18" data-line-end="47" class="language-sh">apiVersion: apps/v1
kind: Deployment
metadata:
  name: bonita-community
spec:
  replicas: <span class="hljs-number">1</span>
  selector:
    matchLabels:
      app: bonita-community
  template:
    metadata:
      labels:
        app: bonita-community
    spec:
      containers:
        - name: bonita
          image: bonita
          env:
            - name: TENANT_LOGIN
              valueFrom:
                secretKeyRef:
                  name: bonita-credentials
                  key: TENANT_LOGIN
            - name: TENANT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bonita-credentials
                  key: TENANT_PASSWORD
</code></pre>
<ul>
<li class="has-line-data" data-line-start="47" data-line-end="48">Create a file named bonita.service1.yaml to expose the Bonita Community deployment.</li>
</ul>
<pre><code class="has-line-data" data-line-start="49" data-line-end="63" class="language-sh">apiVersion: v1
kind: Service
metadata:
  name: bonita-service
spec:
  selector:
    app: bonita-community
  <span class="hljs-built_in">type</span>: NodePort
  ports:
    - protocol: TCP
      port: <span class="hljs-number">8080</span>
      targetPort: <span class="hljs-number">8080</span>
      nodePort: <span class="hljs-number">30000</span>
</code></pre>
<ul>
<li class="has-line-data" data-line-start="63" data-line-end="64">Deploy the updated configuration</li>
</ul>
<pre><code class="has-line-data" data-line-start="65" data-line-end="68" class="language-sh">kubectl apply <span class="hljs-operator">-f</span> bonita-deployment1.yaml
kubectl apply <span class="hljs-operator">-f</span> bonita-service1.yaml
</code></pre>
<h3 class="code-line" data-line-start=69 data-line-end=70 ><a id="Using_PostgreSQL_as_the_database_service_69"></a>Using PostgreSQL as the database service</h3>
<p class="has-line-data" data-line-start="70" data-line-end="71">we use the Pre-configured Postgres database image of Bonita</p>
<pre><code class="has-line-data" data-line-start="72" data-line-end="79" class="language-sh">docker run <span class="hljs-operator">-d</span> 
--name bonita-postgres -p <span class="hljs-number">5432</span>:<span class="hljs-number">5432</span> 
<span class="hljs-operator">-e</span> <span class="hljs-string">"POSTGRES_PASSWORD=bpm"</span> 
<span class="hljs-operator">-e</span> <span class="hljs-string">"POSTGRES_USER=bonita"</span> 
<span class="hljs-operator">-e</span> <span class="hljs-string">"POSTGRES_DB=bonita"</span> 
bonitasoft/bonita-postgres:<span class="hljs-number">15.3</span>
</code></pre>
<ul>
<li class="has-line-data" data-line-start="79" data-line-end="80">Create a Kubernetes Secret to secure login credentials</li>
</ul>
<pre><code class="has-line-data" data-line-start="81" data-line-end="83" class="language-sh">kubectl create secret generic postgres-credentials --from-literal=TENANT_LOGIN=&lt;your-secured-tenant-login&gt; --from-literal=TENANT_PASSWORD=&lt;your-secured-tenant-password&gt;
</code></pre>
<ul>
<li class="has-line-data" data-line-start="83" data-line-end="84">Create a file named bonita-deployment2.yaml and add the following content</li>
</ul>
<pre><code class="has-line-data" data-line-start="85" data-line-end="130" class="language-sh">apiVersion: apps/v1
kind: Deployment
metadata:
  name: bonita-deployment
  labels:
    app: bonita-<span class="hljs-number">2</span>
spec:
  replicas: <span class="hljs-number">1</span>
  selector:
    matchLabels:
      app: bonita-<span class="hljs-number">2</span>
  template:
    metadata:
      labels:
        app: bonita-<span class="hljs-number">2</span>
    spec:
      containers:
      - name: bonita
        image: bonita
        ports:
          - containerPort: <span class="hljs-number">8082</span>
        env:
        - name: DB_VENDOR
          value: postgres
        - name: DB_HOST
          value: localhost
        - name: DB_PORT
          value: <span class="hljs-number">5432</span>
        - name: DB_NAME
          value: bonita
        - name: DB_USER
          value: bonita
        - name: DB_PASS
          value: bpm
        - name: TENANT_LOGIN
          valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: TENANT_LOGIN
        - name: TENANT_PASSWORD
          valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: TENANT_PASSWORD
</code></pre>
<ul>
<li class="has-line-data" data-line-start="130" data-line-end="131">Create a file named bonita.service2.yaml to expose the Bonita Community deployment.</li>
</ul>
<pre><code class="has-line-data" data-line-start="132" data-line-end="146" class="language-sh">apiVersion: v1
kind: Service
metadata:
  name: bonita-service
spec:
  selector:
    app: bonita
  <span class="hljs-built_in">type</span>: LoadBalancer
  ports:
    - protocol: TCP
      port: <span class="hljs-number">8082</span>
      targetPort: <span class="hljs-number">8082</span>
      nodePort: <span class="hljs-number">30001</span>
</code></pre>
<ul>
<li class="has-line-data" data-line-start="146" data-line-end="147">Deploy the updated configuration</li>
</ul>
<pre><code class="has-line-data" data-line-start="148" data-line-end="151" class="language-sh">kubectl apply <span class="hljs-operator">-f</span> bonita-deployment2.yaml
kubectl apply <span class="hljs-operator">-f</span> bonita-service2.yaml
</code></pre>
<h2 class="code-line" data-line-start=152 data-line-end=153 ><a id="User_Story_2_152"></a>User Story 2</h2>
<ul>
<li class="has-line-data" data-line-start="153" data-line-end="154">Create a shell file named log_config.sh</li>
</ul>
<pre><code class="has-line-data" data-line-start="155" data-line-end="160" class="language-sh"><span class="hljs-shebang">#!/bin/bash</span>
<span class="hljs-built_in">echo</span> <span class="hljs-string">"Setting log verbosity to INFO"</span>
xml_file=<span class="hljs-string">"/opt/bonita/conf/logs/log4j2-loggers.xml"</span>
sed -i <span class="hljs-string">'s/level="WARN"/level="INFO"/'</span> <span class="hljs-string">"<span class="hljs-variable">$xml_file</span>"</span>
</code></pre>
<pre><code class="has-line-data" data-line-start="161" data-line-end="163" class="language-sh">kubectl <span class="hljs-built_in">exec</span> &lt;pod-name&gt; -- /bin/sh
</code></pre>
