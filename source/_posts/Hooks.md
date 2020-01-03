---
title: Hooks
date: 2020-01-03 08:56:10
categories: 
- React
tags: 
- React
---
# React Hooks

```
import React , { useState , useEffect } from 'react'


const Demo = () => {
    const [count,setCount] = useState(0)

    useEffect(() => {
        document.title = `clicked ${ count } times`
    })

    return(
        <>
            <p>clicked { count } times</p>
            <button onClick={() => setCount(count + 1)}>
                click me                
            </button>
        </>
    )
}


export default Demo
```

