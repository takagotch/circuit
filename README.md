### circuit
---
https://github.com/cep21/circuit


```go
h := circuit.Manager{}

c := h.MustCreateCircuit("hello-world")

errResult := c.Execute(context.Background(), func(ctx context.Context) error {
  return nil
}, nil)
fmt.Println("Result of execution:", errResult)

c := circuit.NewCircuitFromConfig("hello-world-fallback", circuit.Config{})
errResult := c.Execute(context.Background(), func(ctx context.Context) error {
  return errors.New("this will fail")
}, func(ctx context.Context, err error) error {
  fmt.Println("Circuit failed with error, but fallback returns nil")
  return nil
})
fmt.Println("Execution result:", errResult)


h := circuit.Manager{}
c := h.MustCreateCircuit("untrusting-circuit", circuit.Config{
  Execution: circuit.ExecutionConfig{
    Timeout: time.Millisecond * 30,
  },
})

errResult := c.Go(context.Background(), func(ctx context.Context) error {
  time.Sleep(time.Second * 30)
  return nil
}, nil)
fmt.Printf("err=%v", errResult)


configuration := hystrix.ConfigFactory{
  ConfigureOpener: hystrix.ConfigureOpener{
    RequestVolumeThreshold: 10,
  },
  ConfigureCloser: hystrix.ConfigureCloser{
  },
}
h := circuit.Manager{
  DefaultCircuitProperties: []circuit.CommandPropertiesConstructor{configuration.Configure},
}

c := h.MustCreateCircuit("hystrix-circuit")



sf := rolling.StatFactory{}
h := circuit.Manager{
  DefaultCircuitProperties: []circuit.CommandPropertiesConstructor{sf.CreateConfig},
}
es := metriceventstream.MetricEventStream{
  Manager: &h,
}
go func() {
  if err := es.Start(); err != nil {
    log.Fatal(err)
  }
}()

http.handler("/hystrix.stream", &es)

if err := es.Close(); err != nil {
  log.Fatal(err)
}


h := circuit.Manager{}
expvar.Publish("hystrix", h.Var())


config := circuit.Config{
  Metrics: circuit.MetricsCollectors{
    Run: []circuit.RunMetrics{
    },
  }
}
circuit.NewCircuitFromConfig("custom-metrics", config)


h := circuit.Manager{}
c := h.MustCreateCircuit("panic_up")

defer func() {
  r := recover()
  if r != nil {
    fmt.Println("I recovered from a panic", r)
  }
}()
c.Execute(context.Background(), func(ctx context.Context) error {
  panic("oh no")
}, nil)


configuration := hystrix.ConfigFactory{}
h := circuit.Manager{
  DefaultCircuitProperties: []circuit.CommandPropertiesConstructor{configuration.Configure},
}
c := h.MustCreateCircuit("hystrix-circuit")
fmt.Println("The default sleep window", c.OpenToClose.(*hystrix.Closer).Config().SleepWindow)
c.OpenToClose.(*hystrix.Closer).SetConfigThreadSafe(hystrix.ConfigureCloser{
  SleepWindow: time.Second * 3,
})
fmt.Pringln("The new sleep window", c.OpenToClose.(*hystrix.Closer).Config().SleepWindow)


f := roling.StatFactory{}
manager := circuit.Manager{
  DefaultCircuitProperties: []circuit.CommandPropertiesConstructor{
    f.CreateConfig
  },
}
c := manager.MustCreateCircuit("don't fail me bro")

ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond)
defer cancel()
errResult := c.Execute(ctx, func(ctx context.Context) error {
  select {
  case <- ctx.Done():
    return ctx.Err()
  case <- time.After(time.Hour):
    panic("We never actually get this far")
  }
}, nil)
rs := f.Runstts("don't fail me bro")
fmt.Println("errResult is", errResult)
fmt.Println("The error and timeout count is", rs.ErrTimeouts.TotalSum() + rs.ErrFailures.TotalSum())


myFactory := func(circuitname string) circuit.Config {
  timeoutsByName := map[string]time.Duration{
    "v1": time.Second,
    "v2": time.Second * 2,
  }
  customTimeout := timeoutsByName[circuitName]
  if customTimeout == 0 {
    return circuit.Config{}
  }
  return circuit.Config{
    Execution: circuit.ExecutionConfig{
      Timeout: cutomTimeout,
    },
  }
}

h := circuit.Manager{
  DefaultCircuitProperties: []circuit.CommandPropertiesConstructor{myFactory},
}
h.MustCreateCircuit("v1")
fmt.Println("The timeout o f v1 is", h.GetCircuit("v1").Config().Execution.Timeout)

// StatsD configuration factory
















```

```
```

```
```


