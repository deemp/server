# Notes

## /infer -> execute

request_id
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/python/tritonserver/_api/_request.py#L66>

becomes

id_
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/infer_response.h#L337>

The /infer endpoint
<https://github.com/deemp/server/blob/d7dc5b4c8709eabd336b806fcabae518e0f2a907/src/http_server.cc#L4599>

HandleInfer
<https://github.com/deemp/server/blob/d7dc5b4c8709eabd336b806fcabae518e0f2a907/src/http_server.cc#L3568>

TRITONSERVER_ServerInferAsync
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/include/triton/core/tritonserver.h#L2583>

TRITONSERVER_ServerInferAsync
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/tritonserver.cc#L3230>

InferenceServer::InferAsync
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/server.cc#L537>

InferenceRequest::Run
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/infer_request.cc#L415>

Now, the request is in a queue. Th
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/infer_request.cc#L415>

The queue type depends on the scheduler used in `model_raw_`.
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/infer_request.h#L772>

The scheduler's `Enqueue` is called
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/model.h#L225>

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/model.h#L262>

We assume the dynamic batch scheduler.
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L182>

The batcher adds the `InferenceRequest` to a queue
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L258>

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.h#L146>

and notifies the signaling scheduler thread.

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.h#L154>

There's a batcher thread.

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L299>

It takes requests from the queue
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L389>

Creates a callback in the payload that notifies a thread

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L429>

And adds them to payload
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L394>

Then, it enqueues the payload for a specific model
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/dynamic_batch_scheduler.cc#L434>

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/rate_limiter.cc#L242>

Rate limiter schedules payload
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/rate_limiter.cc#L284>

It puts the payload into an InstanceQueue
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/rate_limiter.cc#L544>

InstanceQueue::Enqueue
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/instance_queue.cc#L52>

payload_queue_
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/instance_queue.h#L57>

in InstanceQueue
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/instance_queue.h#L37>

Also, the Rate Limiter has `DequeuePayload` that dequeues from an InstanceQueue
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/rate_limiter.cc#L348>

Next, it runs a callback
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/rate_limiter.cc#L355>

Maybe this callback notifies the `TritonModelInstance::TritonBackendThread::BackendThread`.

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/backend_model_instance.cc#L768>

There happens `DequeuePayload`
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/backend_model_instance.cc#L787>

And the payload runs `Execute`
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/backend_model_instance.cc#L790>

That may run `Schedule` on the model instance in that payload.
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/payload.cc#L202>

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/payload.cc#L202>

In `Schedule`, `InferenceRequest`s are executed.
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/backend_model_instance.cc#L586>

<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/backend_model_instance.cc#L669>

In `TritonModelInstance::Execute`, the requests are processed using a model exec function.
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/src/backend_model_instance.cc#L679>

And we don't know what happens to requests after that :|.
Probably it's backend-dependent.

TODO:

- Get inference statistics from a backend.
<https://github.com/triton-inference-server/core/blob/bbcd7816997046821f9d1a22e418acb84ca5364b/include/triton/core/tritonbackend.h#L1309-L1359>

- Or, check the code around `TRITON_ENABLE_METRICS_GPU`.
<https://github.com/deemp/server/blob/5ce0d96fec234af5d81307d0bc63ae90a9d5c807/CMakeLists.txt#L60>

- Find out how these statistics get into a response.

