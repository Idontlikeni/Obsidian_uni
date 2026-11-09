1) Проверка softflowd![[Pasted image 20261106190809.png]]
2) Установка контейнера![[Pasted image 20261106191112.png]]
3) Захват flow-записей![[Pasted image 20261106211536.png]]
4) <скрипт агрегации>
5) curl metrics![[Pasted image 20261106212731.png]]
6) prometheus![[Pasted image 20261106213110.png]]
7) панель top talkers![[Pasted image 20261106214116.png]]
8) Unique destinations![[Pasted image 20261106214341.png]]
9) Стырить скрины
10) ![[Pasted image 20261107174053.png]]

flow_top_exporter.py (скрипт агрегации):
```
#!/usr/bin/env python3
import json
import time
import threading
from collections import defaultdict
from http.server import BaseHTTPRequestHandler, HTTPServer

FLOW_LOG = "/var/log/goflow2/flows.log"
TOP_N = 10
lock = threading.Lock()
bytes_by_pair = defaultdict(int)
dst_by_src = defaultdict(set)


def tail_flows():
    with open(FLOW_LOG) as f:
        f.seek(0, 2)
        while True:
            line = f.readline()
            if not line:
                time.sleep(0.5)
                continue
            try:
                rec = json.loads(line)
            except json.JSONDecodeError:
                continue
            src = rec.get("src_addr")
            dst = rec.get("dst_addr")
            nbytes = rec.get("bytes", 0)
            if src and dst:
                with lock:
                    bytes_by_pair[(src, dst)] += nbytes
                    dst_by_src[src].add(dst)


class MetricsHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path != "/metrics":
            self.send_response(404)
            self.end_headers()
            return
        lines = []
        with lock:
            top = sorted(bytes_by_pair.items(), key=lambda x: -x[1])[:TOP_N]
            for (src, dst), nbytes in top:
                lines.append(
                    f'top_talker_bytes_total{{src="{src}",dst="{dst}"}} {nbytes}'
                )
            for src, dsts in dst_by_src.items():
                lines.append(
                    f'flow_unique_destinations{{src="{src}"}} {len(dsts)}'
                )
        body = "\n".join(lines) + "\n"
        self.send_response(200)
        self.send_header("Content-Type", "text/plain; version=0.0.4")
        self.end_headers()
        self.wfile.write(body.encode())


if __name__ == "__main__":
    threading.Thread(target=tail_flows, daemon=True).start()
    HTTPServer(("0.0.0.0", 9200), MetricsHandler).serve_forever()
```