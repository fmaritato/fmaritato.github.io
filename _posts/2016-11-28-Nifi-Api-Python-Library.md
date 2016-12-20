---
layout: single
title: Nifi API Python Library
---

# Introduction
While developing some automation for the Apache Nifi API, I decided it would be easiest to abstract a bunch of repeated functionality
into a library so I didn't have to keep writing the same piece of code over and over. My library uses the requests python library and
demonstrates how to use get, put, post and delete along with some convenience functions.

# Code
Let's start with the basics: get, put, post, delete:

```python
    def remote_put_data(self, path, data):
        response = requests.put(self.url + path, json=data, headers={'Accept': 'application/json',
                                                                     'Content-Type': 'application/json'})
        # Sometimes it returns 201 (created) or 200
        if response.status_code > 299:
            self.logger.warn('POST Error. Status code {} returned. Message {}'.format(response.status_code,
                                                                                      response.text))
            return None
        else:
            return response.json()

    def remote_post_data(self, path, data):
        response = requests.post(self.url + path, json=data)

        # Sometimes it returns 201 (created) or 200
        if response.status_code > 299:
            self.logger.warn('POST Error. Status code {} returned.'.format(response.status_code))
            return None
        else:
            return response

    def remote_post(self, url, filename, accept_mime_type):
        if accept_mime_type is None:
            accept_mime_type = 'application/json'
        response = requests.post(url,
                                 files={'template': open(filename, 'rb')},
                                 headers={'Accept': accept_mime_type})

        # Sometimes it returns 201 (created) or 200
        if response.status_code > 299:
            self.logger.debug('POST Error. Status code {} returned.'.format(response.status_code))
            return None
        else:
            return response

    def remote_delete(self, path, id):
        if id is None:
            response = requests.delete(self.url + path)
        else:
            response = requests.delete(self.url + path + id)
        if response.status_code != 200:
            self.logger.debug('DELETE Error. Status code {} returned.'.format(response.status_code))
            return None
        else:
            return response
            
    def remote_get(self, path, id):
        if id is None:
            response = requests.get(self.url + path, headers={'Accept': 'application/json'})
        else:
            response = requests.get(self.url + path + id, headers={'Accept': 'application/json'})
        if response.status_code != 200:
            self.logger.debug('GET Error. Status code {} returned.'.format(response.status_code))
            return None
        else:
            return response.json()            
```