# sysmon
## 준비물
1. Source Host: 모니터링 대상 Host
2. Metric Crawler/forwarder : 대상 Host의 Metric을 생성하는 deamon(CollectD)
  - CollectD
3. Metric Aggregator : 수집되는 Metric을 Metric Repository에 저장하는 Queue(Deffered, Async) 또는 Metirc의 데이터 정제 역할
  - CollectD
4. Metric Repository : 시간과 Host를 기준으로 Metirc을 저장해 주는 시계열DB
  - InfluxDB
5. Metric Dashboard : Metric Repositoy를 DataSource로 사용하여 시계열 그래프로 Metric정보를 표시해 주는 역할
  - Grafana
