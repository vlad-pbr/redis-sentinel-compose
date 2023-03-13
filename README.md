Redis + Sentinel Compose File
=============================

Made to be deployed on Docker Swarm. Dirty, but practical.

1. Replace `192.168.1.75` with your environment's address (load balancer between nodes, a VIP, etc.)

2. Deploy via `docker stack deploy`.

3. Redis master can now be retrieved via following redis sentinels:
   - `{environment-ip}:26371`
   - `{environment-ip}:26372`
   - `{environment-ip}:26373`

4. ???

5. PROFIT