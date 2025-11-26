## More About Me – [Take a Look!](http://www.mjakaria.me)

### Welcome to [Prometheus](https://prometheus.io/docs/introduction/overview/)

Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community. It is now a standalone open source project and maintained independently of any company. To emphasize this, and to clarify the project's governance structure, Prometheus joined the Cloud Native Computing Foundation in 2016 as the second hosted project, after Kubernetes.

#### [Prerequisites](https://training.promlabs.com/training/introduction-to-prometheus/training-overview/prerequisites)

- Ubuntu Linux 20.04 or Upper
- Basic knowledge of Unix / Linux server administration.

#### [Installation](https://prometheus.io/download/)

```bash
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0-rc.0/prometheus-2.52.0-rc.0.linux-arm64.tar.gz
tar xzvf prometheus-2.52.0-rc.0.linux-arm64.tar.gz
mv prometheus-2.52.0-rc.0.linux-arm64 prometheus-2.52
rm -f prometheus-2.52.0-rc.0.linux-arm64.tar.gz
cd prometheus-2.52
ls -l
rm -f console* LICENSE NOTICE # remove unnecessary files [optional]
rm promtool # remove unnecessary directories [optional]
./prometheus
http://localhost/IP:9090
```

[Architecture](https://prometheus.io/docs/introduction/overview/)
![Architecture](/img/architecture.png 'Architecture')

## With Regards, `Jakir`

[![LinkedIn][linkedin-shield-jakir]][linkedin-url-jakir]
[![Facebook-Page][facebook-shield-jakir]][facebook-url-jakir]
[![Youtube][youtube-shield-jakir]][youtube-url-jakir]

### Wishing you a wonderful day! Keep in touch

<!-- Personal profile -->

[linkedin-shield-jakir]: https://img.shields.io/badge/linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white
[linkedin-url-jakir]: https://www.linkedin.com/in/jakir-ruet/
[facebook-shield-jakir]: https://img.shields.io/badge/Facebook-%231877F2.svg?style=for-the-badge&logo=Facebook&logoColor=white
[facebook-url-jakir]: https://www.facebook.com/jakir.ruet/
[youtube-shield-jakir]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=for-the-badge&logo=YouTube&logoColor=white
[youtube-url-jakir]: https://www.youtube.com/@mjakaria-ruet/featured
