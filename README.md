# VRPTW Instances
## Overview
This dataset utilize knowledge from analyzing real sample data and collected data about Bangkok. The sample data studied is order and delivery data from a company in the United States published by [ORTEC](https://github.com/ortec/euro-neurips-vrp-2022-quickstart/tree/main/instances). The Bangkok data includes population and geographic information. I created a total of 365 instances simulating a supermarket business operating for 365 days. The data was modeled to have characteristics and distributions matching the sample data, with relationships between variables made realistic to the Bangkok context. Customer type and instance day variables were added and related to other variables.
## Usage
To read these instances, make sure use my `read_vrplib` function.

The original function is from [EURO Meets NeurIPS 2022 Vehicle Routing Competition](https://github.com/ortec/euro-neurips-vrp-2022-quickstart/blob/main/tools.py)

``` python
def read_vrplib(filename, rounded=True):
    loc = []
    unit = []
    demand = []
    mode = ''
    dimension = None
    capacity = None
    vehicle = None
    day = None
    edge_weight_type = None
    edge_weight_format = None
    duration_matrix = []
    service_t = []
    timewi = []
    with open(filename, 'r') as f:

        for line in f:
            line = line.strip(' \t\n')
            if line == "":
                continue
            elif line.startswith('DIMENSION'):
                dimension = int(line.split(" : ")[1])
            elif line.startswith('CAPACITY'):
                capacity = int(line.split(" : ")[1])
            elif line.startswith('VEHICLES'):
                vehicle = int(line.split(" : ")[1])
            elif line.startswith('DAYS'):
                day = str(line.split(" : ")[1])
            elif line.startswith('EDGE_WEIGHT_TYPE'):
                edge_weight_type = line.split(" : ")[1]
            elif line.startswith('EDGE_WEIGHT_FORMAT'):
                edge_weight_format = line.split(" : ")[1]
            elif line == 'NODE_COORD_SECTION':
                mode = 'coord'
            elif line == 'REQUEST_CUSTOMER_TYPE_SECTION':
                mode = 'unit'
            elif line == 'DEMAND_SECTION':
                mode = 'demand'
            elif line == 'DEPOT_SECTION':
                mode = 'depot'
            elif line == "EDGE_WEIGHT_SECTION":
                mode = 'edge_weights'
                assert edge_weight_type == "EXPLICIT"
                assert edge_weight_format == "FULL_MATRIX"
            elif line == "TIME_WINDOW_SECTION":
                mode = "time_windows"
            elif line == "SERVICE_TIME_SECTION":
                mode = "service_t"
            elif line == "EOF":
                break
            elif mode == 'coord':
                node, x, y = line.split()  # Split by whitespace or \t, skip duplicate whitespace
                node = int(node)
                x, y = (int(x), int(y)) if rounded else (float(x), float(y))

                if node == 1:
                    depot = (x, y)
                else:
                    assert node == len(loc) + 2 # 1 is depot, 2 is 0th location
                    loc.append((x, y))
            elif mode == 'unit':
                node, u = [str(v) for v in line.split()]
                if node == 1:
                    assert u == 'depot'
                unit.append(u)
            elif mode == 'demand':
                node, d = [int(v) for v in line.split()]
                if node == 1:
                    assert d == 0
                demand.append(d)
            elif mode == 'edge_weights':
                duration_matrix.append(list(map(int if rounded else float, line.split())))
            elif mode == 'service_t':
                node, t = line.split()
                node = int(node)
                t = int(t) if rounded else float(t)
                if node == 1:
                    assert t == 0
                assert node == len(service_t) + 1
                service_t.append(t)
            elif mode == 'time_windows':
                node, l, u = line.split()
                node = int(node)
                l, u = (int(l), int(u)) if rounded else (float(l), float(u))
                assert node == len(timewi) + 1
                timewi.append([l, u])

    return {
        'is_depot': np.array([1] + [0] * len(loc), dtype=bool),
        'coords': np.array([depot] + loc),
        'units': np.array(unit),
        'demands': np.array(demand),
        'dimension': dimension,
        'capacity': capacity,
        'vehicle': vehicle,
        'day': day,
        'time_windows': np.array(timewi),
        'service_times': np.array(service_t),
        'duration_matrix': np.array(duration_matrix) if len(duration_matrix) > 0 else None
    }
```
