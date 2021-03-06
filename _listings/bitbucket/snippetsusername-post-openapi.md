---
swagger: "2.0"
x-collection-name: Bitbucket
x-complete: 0
info:
  title: Bitbucket Add Snippets Username
  description: |-
    Identical to `/snippets`, except that the new snippet will be
    created under the account specified in the path parameter `{username}`.
  termsOfService: https://www.atlassian.com/legal/customer-agreement
  contact:
    name: Bitbucket Support
    url: https://support.atlassian.com/bitbucket
    email: support@bitbucket.org
  version: 1.0.0
host: api.bitbucket.org
basePath: /2.0
schemes:
- http
produces:
- application/json
consumes:
- application/json
paths:
  /snippets:
    get:
      summary: Get Snippets
      description: |-
        Returns all snippets. Like pull requests, repositories and teams, the
        full set of snippets is defined by what the current user has access to.

        This includes all snippets owned by the current user, but also all snippets
        owned by any of the teams the user is a member of, or snippets by other
        users that the current user is either watching or has collaborated on (for
        instance by commenting on it).

        To limit the set of returned snippets, apply the
        `?role=[owner|contributor|member]` query parameter where the roles are
        defined as follows:

        * `owner`: all snippets owned by the current user
        * `contributor`: all snippets owned by, or watched by the current user
        * `member`: owned by the user, their teams, or watched by the current user

        When no role is specified, all public snippets are returned, as well as all
        privately owned snippets watched or commented on.

        The returned response is a normal paginated JSON list. This endpoint
        only supports `application/json` responses and no
        `multipart/form-data` or `multipart/related`. As a result, it is not
        possible to include the file contents.
      operationId: getSnippets
      x-api-path-slug: snippets-get
      parameters:
      - in: query
        name: role
        description: Filter down the result based on the authenticated users role
          (`owner`, `contributor`, or `member`)
      responses:
        200:
          description: OK
      tags:
      - Snippets
    parameters:
      summary: Parameters Snippets
      description: Parameters snippets
      operationId: parametersSnippets
      x-api-path-slug: snippets-parameters
      responses:
        200:
          description: OK
      tags:
      - Snippets
    post:
      summary: Add Snippets
      description: |-
        Creates a new snippet under the authenticated user's account.

        Snippets can contain multiple files. Both text and binary files are
        supported.

        The simplest way to create a new snippet from a local file:

            $ curl -u username:password -X POST https://api.bitbucket.org/2.0/snippets               -F file=@image.png

        Creating snippets through curl has a few limitations and so let's look
        at a more complicated scenario.

        Snippets are created with a multipart POST. Both `multipart/form-data`
        and `multipart/related` are supported. Both allow the creation of
        snippets with both meta data (title, etc), as well as multiple text
        and binary files.

        The main difference is that `multipart/related` can use rich encoding
        for the meta data (currently JSON).


        multipart/related (RFC-2387)
        ----------------------------

        This is the most advanced and efficient way to create a paste.

            POST /2.0/snippets/evzijst HTTP/1.1
            Content-Length: 1188
            Content-Type: multipart/related; start="snippet"; boundary="===============1438169132528273974=="
            MIME-Version: 1.0

            --===============1438169132528273974==
            Content-Type: application/json; charset="utf-8"
            MIME-Version: 1.0
            Content-ID: snippet

            {
              "title": "My snippet",
              "is_private": true,
              "scm": "hg",
              "files": {
                  "foo.txt": {},
                  "image.png": {}
                }
            }

            --===============1438169132528273974==
            Content-Type: text/plain; charset="us-ascii"
            MIME-Version: 1.0
            Content-Transfer-Encoding: 7bit
            Content-ID: "foo.txt"
            Content-Disposition: attachment; filename="foo.txt"

            foo

            --===============1438169132528273974==
            Content-Type: image/png
            MIME-Version: 1.0
            Content-Transfer-Encoding: base64
            Content-ID: "image.png"
            Content-Disposition: attachment; filename="image.png"

            iVBORw0KGgoAAAANSUhEUgAAABQAAAAoCAYAAAD+MdrbAAABD0lEQVR4Ae3VMUoDQRTG8ccUaW2m
            TKONFxArJYJamCvkCnZTaa+VnQdJSBFl2SMsLFrEWNjZBZs0JgiL/+KrhhVmJRbCLPx4O+/DT2TB
            cbblJxf+UWFVVRNsEGAtgvJxnLm2H+A5RQ93uIl+3632PZyl/skjfOn9Gvdwmlcw5aPUwimG+NT5
            EnNN036IaZePUuIcK533NVfal7/5yjWeot2z9ta1cAczHEf7I+3J0ws9Cgx0fsOFpmlfwKcWPuBQ
            73Oc4FHzBaZ8llq4q1mr5B2mOUCt815qYR8eB1hG2VJ7j35q4RofaH7IG+Xrf/PfJhfmwtfFYoIN
            AqxFUD6OMxcvkO+UfKfkOyXfKdsv/AYCHMLVkHAFWgAAAABJRU5ErkJggg==
            --===============1438169132528273974==--

        The request contains multiple parts and is structured as follows.

        The first part is the JSON document that describes the snippet's
        properties or meta data. It either has to be the first part, or the
        request's `Content-Type` header must contain the `start` parameter to
        point to it.

        The remaining parts are the files of which there can be zero or more.
        Each file part should contain the `Content-ID` MIME header through
        which the JSON meta data's `files` element addresses it. The value
        should be the name of the file.

        `Content-Disposition` is an optional MIME header. The header's
        optional `filename` parameter can be used to specify the file name
        that Bitbucket should use when writing the file to disk. When present,
        `filename` takes precedence over the value of `Content-ID`.

        When the JSON body omits the `files` element, the remaining parts are
        not ignored. Instead, each file is added to the new snippet as if its
        name was explicitly linked (the use of the `files` elements is
        mandatory for some operations like deleting or renaming files).


        multipart/form-data
        -------------------

        The use of JSON for the snippet's meta data is optional. Meta data can
        also be supplied as regular form fields in a more conventional
        `multipart/form-data` request:

            $ curl -X POST -u credentials https://api.bitbucket.org/2.0/snippets               -F title="My snippet"               -F file=@foo.txt -F file=@image.png

            POST /2.0/snippets HTTP/1.1
            Content-Length: 951
            Content-Type: multipart/form-data; boundary=----------------------------63a4b224c59f

            ------------------------------63a4b224c59f
            Content-Disposition: form-data; name="file"; filename="foo.txt"
            Content-Type: text/plain

            foo

            ------------------------------63a4b224c59f
            Content-Disposition: form-data; name="file"; filename="image.png"
            Content-Type: application/octet-stream

            ?PNG

            IHDR?1??I.....
            ------------------------------63a4b224c59f
            Content-Disposition: form-data; name="title"

            My snippet
            ------------------------------63a4b224c59f--

        Here the meta data properties are included as flat, top-level form
        fields. The file attachments use the `file` field name. To attach
        multiple files, simply repeat the field.

        The advantage of `multipart/form-data` over `multipart/related` is
        that it can be easier to build clients.

        Essentially all properties are optional, `title` and `files` included.


        Sharing and Visibility
        ----------------------

        Snippets can be either public (visible to anyone on Bitbucket, as well
        as anonymous users), or private (visible only to the owner, creator
        and members of the team in case the snippet is owned by a team). This
        is controlled through the snippet's `is_private` element:

        * **is_private=false** -- everyone, including anonymous users can view
          the snippet
        * **is_private=true** -- only the owner and team members (for team
          snippets) can view it

        To create the snippet under a team account, just append the team name
        to the URL (see `/2.0/snippets/{username}`).
      operationId: postSnippets
      x-api-path-slug: snippets-post
      parameters:
      - in: body
        name: _body
        description: The new snippet object
        schema:
          $ref: '#/definitions/holder'
      responses:
        200:
          description: OK
      tags:
      - Snippets
  /snippets/{username}:
    get:
      summary: Get Snippets Username
      description: |-
        Identical to `/snippets`, except that the result is further filtered
        by the snippet owner and only those that are owned by `{username}` are
        returned.
      operationId: getSnippetsUsername
      x-api-path-slug: snippetsusername-get
      parameters:
      - in: query
        name: role
        description: Filter down the result based on the authenticated users role
          (`owner`, `contributor`, or `member`)
      - in: path
        name: username
        description: Limits the result to snippets owned by this user
      responses:
        200:
          description: OK
      tags:
      - Snippets
      - Username
    parameters:
      summary: Parameters Snippets Username
      description: Parameters snippets username
      operationId: parametersSnippetsUsername
      x-api-path-slug: snippetsusername-parameters
      responses:
        200:
          description: OK
      tags:
      - Snippets
      - Username
    post:
      summary: Add Snippets Username
      description: |-
        Identical to `/snippets`, except that the new snippet will be
        created under the account specified in the path parameter `{username}`.
      operationId: postSnippetsUsername
      x-api-path-slug: snippetsusername-post
      parameters:
      - in: body
        name: _body
        description: The new snippet object
        schema:
          $ref: '#/definitions/holder'
      responses:
        200:
          description: OK
      tags:
      - Snippets
      - Username
x-streamrank:
  polling_total_time_average: 0
  polling_size_download_average: 0
  streaming_total_time_average: 0
  streaming_size_download_average: 0
  change_yes: 0
  change_no: 0
  time_percentage: 0
  size_percentage: 0
  change_percentage: 0
  last_run: ""
  days_run: 0
  minute_run: 0
---