# Uploading user files into a Lyra workflow

`create_upload_image_node`, `create_upload_video_node`, `create_upload_audio_node` and
`create_upload_documents_node` take a `file_url`. The server downloads the file and stores it in
Lyra, then adds an upload node that holds it. Connect that node like any generated one.

## What the URL must be

- `https://`, on a public internet host. Private IPs, localhost and VPN-only hosts are rejected.
- **No redirects.** Share links that redirect (many Drive/Dropbox "view" links) fail. Ask for the direct
  file link, for example Dropbox `?dl=1` or a CDN URL.
- Opens without signing in. A link behind a login returns an error page, not the file.
- ≤ 100 MB, and the download must finish within about 60 s.
- The file type must match the tool. An image sent to `create_upload_video_node` is rejected. Documents
  accept PDF, Office/ODF, CSV, TXT, Markdown, JSON, XML, RTF and EPUB.

## Files on the user's machine

The server cannot read local paths. If the user has only a local file, ask them to put it somewhere
that gives a public direct link. They can also add it on the Lyra canvas themselves. Then
`get_workflow` gives you the uuid of the upload node they added.

## Gotchas

- Lyra cannot delete a stored upload. Do not upload the same file again to "retry" a node that failed
  for an unrelated reason. Reuse the existing upload node.
- An uploaded image used as a video `first_frame` / `last_frame` behaves like a generated one: connect
  it after it exists. Upload nodes hold their file as soon as they are created.
