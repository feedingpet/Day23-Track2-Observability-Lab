# HELP python_gc_objects_collected_total Objects collected during gc
# TYPE python_gc_objects_collected_total counter
python_gc_objects_collected_total{generation="0"} 2139.0
python_gc_objects_collected_total{generation="1"} 3109.0
python_gc_objects_collected_total{generation="2"} 616.0
# HELP python_gc_objects_uncollectable_total Uncollectable objects found during GC
# TYPE python_gc_objects_uncollectable_total counter
python_gc_objects_uncollectable_total{generation="0"} 0.0
python_gc_objects_uncollectable_total{generation="1"} 0.0
python_gc_objects_uncollectable_total{generation="2"} 0.0
# HELP python_gc_collections_total Number of times this generation was collected
# TYPE python_gc_collections_total counter
python_gc_collections_total{generation="0"} 152.0
python_gc_collections_total{generation="1"} 13.0
python_gc_collections_total{generation="2"} 1.0
# HELP python_info Python platform information
# TYPE python_info gauge
python_info{implementation="CPython",major="3",minor="12",patchlevel="13",version="3.12.13"} 1.0
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.104330752e+09
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 7.8405632e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.77847415436e+09
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 5.22
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 20.0
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP inference_requests_total Total inference requests
# TYPE inference_requests_total counter
# HELP inference_latency_seconds Inference end-to-end latency
# TYPE inference_latency_seconds histogram
# HELP inference_active_gauge In-flight inference requests
# TYPE inference_active_gauge gauge
inference_active_gauge 0.0
# HELP inference_tokens_total Tokens processed (input/output)
# TYPE inference_tokens_total counter
# HELP inference_quality_score Latest eval-as-metric quality score [0,1]
# TYPE inference_quality_score gauge
# HELP gpu_utilization_percent Simulated GPU utilization [0,100]
# TYPE gpu_utilization_percent gauge
gpu_utilization_percent 77.01802878282318