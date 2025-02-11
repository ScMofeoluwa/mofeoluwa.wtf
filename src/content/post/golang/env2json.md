---
title: "Env to Json"
description: "A basic golang script to convert your .env variables to json"
publishDate: "14 Mar 2023"
tags: ["golang"]
---


```go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"os"
	"strings"
)

func main() {
	env2json("test.env", "test.json")
}

func env2json(envFilePath string, jsonFilePath string) {
	scanner := bufio.NewScanner(read(envFilePath))
	writer := bufio.NewWriter(write(jsonFilePath))
	res := make(map[string]string)
	for scanner.Scan() {
		line := scanner.Text()
		if strings.HasPrefix(line, "#") {
			continue
		}
		s := strings.Split(line, "=")
		res[s[0]] = s[1]
	}
	obj := serialize(res, "_")
	data, _ := json.MarshalIndent(obj, "", "   ")

	for _, line := range data {
		writer.WriteByte(line)
	}
	writer.Flush()
	fmt.Println("data written")
}

func read(path string) (f *os.File) {
	f, err := os.Open(path)
	if err != nil {
		fmt.Println(err)
	}
	return
}

func write(path string) (f *os.File) {
	f, err := os.Create(path)
	if err != nil {
		fmt.Println(err)
	}
	return
}

func serialize(obj map[string]string, delimiter string) map[string]interface{} {
	output := make(map[string]interface{})
	for key, value := range obj {
		res := make(map[string]string)
		words := strings.Split(key, delimiter)
		if len(words) == 2 {
			res[words[1]] = value
			output[words[0]] = res
		} else {
			output[key] = value
		}
	}
	return output
}
```
