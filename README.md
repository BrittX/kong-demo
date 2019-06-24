# Testing out Kong

### Bring up the containers
`(venv) docker-compose up`

### Add stang service/route to Kong
1. Make sure `stang` is running `(venv) python stang/main.py`
  - Change the host addy to your IP
2. Add the service: `(venv) http :8001/services/ name=vehicle-service url=http://<IP_ADDY>:8002/docs host=192.168.241.131`
3. Add a route for the service: `(venv) http POST :8001/services/vehicle-service/routes`
4. Forward requests through Kong: `(venv) http :8000/docs`


### Access Konga in browser
1. Get the IP of the `konga` container
2. Open browser and head to <IP_ADDY>:1337
3. Add info for adding an admin user
4. Add new connection:
```
Name: kong
Kong admin url: http://kong:8001
```
5. Activate the connection


### Add new service using Konga
1. Select `Services` and then `New Service`
2. Note: Name should be name of our service, host=host IP of the service, port=port used by the service.


### Add new route using Konga
1. While looking at a particular service, select `Routes` then `Add Route`
2. Need to add all the available routes for the service, ex:
```
Name=<name  of route>
Paths=<path matching the route>
Methods=[of methods available on the route]
Protocols=[http, https]
```

### Need to do
#### Automatically register service with KONG
1. When a service or container comes online, its first action should be to register online
  - Check first to register the service, if service exists, then ignore
Solutions:
  - Use a wait for it script that auto registers
  - Use something like `Consul` to handle autodiscover and connecting
2. (Optional) Tag each service with service name
  - Do a look up based on the tag? Then view the routes available?

#### Send items from one service to another when online
1. Service1 periodically checks if ServiceN is online
  - Should always occur when Service comes online
  - Should always occur whenever the service is running
2. If ServiceN online, look up by identifier if items stored in Service1 found in ServiceN
  - if not found, POST items to ServiceN
  - else update items in ServiceN
  #### Approach?
  1. Use background celery task to periodically check for the service?
    - Does this celery task happen in each service? Or just Service1?
  2. Add a script that just runs forever to do this?
    - Put the script in Service1s container and just run it on start?

#### Get api data from each service
1. Custom endpoint to return api data depending on service available
  - ie Have Service1 and Service2
    - Only Service1 available:
      - Check if Service1 available, it is, get endpoint X to return, have this return a `next` with empty list if Service2 not available
      - Check if Service2 available, it's not, return
    - Service1 and Serive2 available
      - Check if Service1 available, it is, get endpoint X's data to return, have this return a `next` with an api to hit containing Service2's api data
      - Check if Service2 available, it is, get endpoint Y's data to return

#### POST/PUT/DELETE/PATCH for each service
1. Based on how previous section works, need to send data to appropriate endpoints/services
  - Have Service1 and Service2
    - Only Service1 available:
      - Once data filled in for Service1, do method against Service1's endpoint
      - Return Service1's response back
    - Both Service1 and Service2 available:
      - Once data filled in for Service1, do method against Service1's endpoint
