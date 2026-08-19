# File Upload

File Upload lets the user attach files, either by dropping them on a drop zone or by picking them from a file dialog. Selected files appear in a queue where they can be removed, renamed and commented on before the form is submitted.

## Availability

| Package        | Available | Tag / Import                                                                                      |
| -------------- | --------- | --------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktFileUpload>` — `import { PktFileUpload } from '@oslokommune/punkt-react'`                    |
| Elements       | Yes       | `<pkt-fileupload>` — `import '@oslokommune/punkt-elements/dist/pkt-fileupload.js'`                |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-fileupload.js" type="module">` |

Dark mode: Yes

## Upload strategies

The `uploadStrategy` prop decides when files leave the browser. Pick one up front — it changes which other props you need.

| Strategy   | Behaviour                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| `"form"`   | Default. Files ride along with the normal form submit via a native `<input type="file">`. No JavaScript upload code needed |
| `"custom"` | Files upload immediately as they are added. You start the upload and report progress back                |

With `"custom"` you must supply `id`, `transfers` and `onFileUploadRequested`. Punkt renders the progress; you do the transferring.

## Props / Attributes

| Prop (React)             | Attribute (Elements)        | Type                                     | Default      | Description                                                              |
| ------------------------ | --------------------------- | ---------------------------------------- | ------------ | ------------------------------------------------------------------------ |
| `name`                   | `name`                      | string                                   | **(required)** / `"files"` | Form field name                                            |
| `id`                     | `id`                        | string                                   | auto         | Required when `uploadStrategy="custom"`                                  |
| `uploadStrategy`         | `upload-strategy`           | `"form"` \| `"custom"`                   | `"form"`     | When files are uploaded                                                  |
| `multiple`               | `multiple`                  | boolean                                  | `false`      | Allow more than one file                                                 |
| `value`                  | `.value`                    | `FileItem[]`                             | —            | Controlled file list                                                     |
| `defaultValue`           | `.defaultValue`             | `FileItem[]`                             | —            | Uncontrolled initial list. Use either this or `value`, not both          |
| `disabled`               | `disabled`                  | boolean                                  | `false`      | Disables the whole component                                             |
| `required`               | `required`                  | boolean                                  | `false`      | Marks the upload as required                                             |
| `fullwidth`              | `fullwidth`                 | boolean                                  | `false`      | Stretch drop zone and queue to full container width                      |
| `accept`                 | `accept`                    | string                                   | `""`         | Native file-dialog filter                                                |
| `allowedFormats`         | `allowed-formats`           | string[]                                 | —            | Validated formats — extensions (`pdf`) or MIME patterns (`image/*`)      |
| `formatErrorMessage`     | `format-error-message`      | string                                   | —            | Message for invalid format. `{formats}` is substituted                   |
| `maxFileSize`            | `max-file-size`             | string \| number                         | —            | Max size, e.g. `"5MB"` or a byte count                                   |
| `sizeErrorMessage`       | `size-error-message`        | string                                   | —            | Message for oversized files. `{maxSize}` is substituted                  |
| `itemRenderer`           | `item-renderer`             | `"filename"` \| `"thumbnail"` \| function | `"filename"` | How queue items are rendered                                             |
| `enableImagePreview`     | `enable-image-preview`      | boolean                                  | `false`      | Image preview modal. Only applies to the thumbnail renderer              |
| `truncateTail`           | `truncate-tail`             | number                                   | `4`          | Trailing characters kept when truncating long filenames                  |
| `addCommentsEnabled`     | `add-comments-enabled`      | boolean                                  | `false`      | Allow commenting on queued files                                         |
| `renameFilesEnabled`     | `rename-files-enabled`      | boolean                                  | `false`      | Allow renaming queued files                                              |
| `extraOperations`        | `.extraOperations`          | `TQueueItemOperation[]`                  | `[]`         | Extra per-file actions, appended after the built-ins                     |
| `transfers`              | `.transfers`                | `TFileTransfer[]`                        | `[]`         | Upload progress. Required for `uploadStrategy="custom"`                  |
| `hasError`               | `has-error`                 | boolean                                  | `false`      | External error flag, combined with internal validation                   |
| `errorMessage`           | `error-message`             | string                                   | —            | External error message shown under the component                         |

Comments and renaming are disabled automatically in the thumbnail view.

The component also accepts the standard Input Wrapper label props — `label`, `helptext`, `helptextDropdown`, `helptextDropdownButton`, `optionalTag`, `optionalText`, `requiredTag`, `requiredText` and `tagText` — which wrap it in a `PktInputWrapper`. See [Input Wrapper](input-wrapper.md).

## Events

| Event (React)            | Event (Elements)         | Description                                                        |
| ------------------------ | ------------------------ | ------------------------------------------------------------------ |
| `onFilesChanged`         | `files-changed`          | The file list changed — files added, removed, renamed or commented |
| `onFileUploadRequested`  | `file-upload-requested`  | A file is ready to upload. Only with `uploadStrategy="custom"`     |
| `onTransferCancelled`    | `transfer-cancelled`     | The user cancelled an in-flight upload                             |
| `onFileValidation`       | —                        | Extra validation hook. Return a string to reject the file          |
| `onFileValidate`         | `file-validate`          | Low-level validation. Set `errorMessage` on the detail to reject   |

`onFileValidation` runs after the built-in format and size checks, so use it for rules Punkt cannot know about.

## Shapes

```ts
interface FileItem {
  fileId: string
  file?: File
  attributes: { targetFilename: string } & Record<string, unknown>
}

interface FileTransfer {
  fileId: string
  progress: TTransferProgress
  errorMessage?: string
  showProgress?: boolean
}
```

## Accessibility

- The drop zone is operable from the keyboard — dropping is never the only way to add a file
- Validation errors are announced, not only shown in colour
- Keep `label` set so the field has an accessible name

## Examples

### React

```jsx
import { PktFileUpload } from '@oslokommune/punkt-react'

{
  /* Form upload — files submit with the form */
}
;<PktFileUpload
  name="attachments"
  label="Attachments"
  helptext="PDF or images, max 5 MB each"
  multiple
  required
  allowedFormats={['pdf', 'image/*']}
  maxFileSize="5MB"
  onFilesChanged={(files) => setFiles(files)}
/>

{
  /* Immediate upload — you transfer the file and report progress */
}
;<PktFileUpload
  id="receipts"
  name="receipts"
  label="Receipts"
  uploadStrategy="custom"
  transfers={transfers}
  onFileUploadRequested={(fileItem) => startUpload(fileItem)}
  onTransferCancelled={(fileItemId) => abortUpload(fileItemId)}
/>
```

### Elements

```html
<pkt-fileupload
  name="attachments"
  label="Attachments"
  helptext="PDF or images, max 5 MB each"
  multiple
  required
  allowed-formats="pdf,image/*"
  max-file-size="5MB"
></pkt-fileupload>

<script>
  const upload = document.querySelector('pkt-fileupload')
  upload.addEventListener('files-changed', (e) => console.log(e.detail))
</script>
```

## Notes

`allowedFormats` is a comma-separated string in the `allowed-formats` attribute, but an array when set as a property. `value`, `defaultValue`, `transfers` and `extraOperations` are property-only — they cannot be set as attributes.

`accept` only filters the native file dialog. It is not validation — use `allowedFormats` for that, since files can still arrive by drag and drop.
