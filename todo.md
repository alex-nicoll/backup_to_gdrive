- Investigate this error that occured when running from a directory other than home:
```
backing up /home/alexn ...
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    File "/usr/lib/python3.5/json/__init__.py", line 268, in load
        parse_constant=parse_constant, object_pairs_hook=object_pairs_hook, **kw)
          File "/usr/lib/python3.5/json/__init__.py", line 319, in loads
              return _default_decoder.decode(s)
                File "/usr/lib/python3.5/json/decoder.py", line 339, in decode
                    obj, end = self.raw_decode(s, idx=_w(s, 0).end())
                      File "/usr/lib/python3.5/json/decoder.py", line 357, in raw_decode
                          raise JSONDecodeError("Expecting value", s, err.value) from None
                          json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
                          * Could not resolve host: www.googleapis.com
                          * Closing connection 0
```
- Investigate whether backup beomes corrupted if we write to a file while tar-ing
- Implement general error handling and integrity checking.
  - Ensure that failure codes are properly reported at each step of pipeline. Retry if pipeline fails.
  - After backup completes, attempt a restore. If it fails, retry backup.
  - A/B backups. Don't lose the previous backup in case of failure.
  - Log errors.
  - Send an email after N failed retries.
- Include API key as a query parameter? Google's "Try it now" example does so, but this seems otherwise undocumented. Check API v2 docs.
- Remove step 4. `backup` should search for file by known appProperty, if it exists update it, else create file with known appProperty.
- Rather than creating cron that runs anacron every 15 minutes, run anacron when computer wakes up.
