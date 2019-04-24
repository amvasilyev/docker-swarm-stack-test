# Running this configuration

1. Install latest release of the docker engire. Currently the test were made with 18.09.5 release.
2. Switch Docker into the swarm mode:

   ```
   docker swarm init --default-addr-poll 10.254.0.0/16
   ```

   You may specify another address poll, if it overlaps with your organization network.

   In order to leave the swarm mode execute the following command.
   ```
   docker swarm leave --force
   ```

3. Configure the routing to the swarm network.
   1. You need to know the IP of the host that provides access to the overlay networks:

      ```
      ip route | grep docker_gwbridge
      ```

      My output is:

      ```
      172.18.0.0/16 dev docker_gwbridge proto kernel scope link src 172.18.0.1
      ```

      The required address is: `172.18.0.1`.
   2. Add route to the swarm address pool:

      ```
      sudo route add -net 10.254.0.0/16 gw 172.18.0.1
      ```

4. Create the stack for the back-end:

   ```
   docker stack deploy -c service.yaml test_000
   ```

   This stack consists out of 4 instances of Mariadb 10.3 with users repl, maxuser created.
5. Get the IP-addresses of the back-end.
   1. Get the id of the stack tasks:

   ```
   docker stack ps test_000
   ```

   2. Get the IP address from the task:

   ```
   docker inspect --format '{{range .NetworksAttachments}}{{.Addresses}}{{end}}' TASK_ID
   ```

6. Spin-up the MaxScale that will connect to the services:

   ```
   docker stack deploy -c full-service.yaml test_000
   ```

7. Inspect the MaxScale startup log
   1. Get the id of the service that belong to required task:

   ```
   docker inspect --format "{{.ServiceID}}" TASK_ID
   ```

   2. Get the logs of the service

   ```
   docker service logs SERVICE_ID
   ```

8. When finished, remove the service stack:

   ```
   docker stack rm test_000
   ```
