# Neural — Smooth-50 reproduction

This package reproduces the organizer's Smooth-50 reference baseline; it is not a trained model or a claim of improvement. The baseline source and game metadata helper are copied from Metrica Sports' More Than A Hack 2027 repository under the included license. Team Neural contributed packaging, reproduction checks and research documented in Neural.md.

The separately licensed match data are not included and must not be published. Obtain the authorized data bundle through the competition; place the tactical ATD text and XML files inside a `tactical` directory under an external data root.

PowerShell, from this directory:

```powershell
$env:MTAH_DATA_DIR = 'C:\path\to\authorized-data-root'
python -m pip install --require-hashes -r requirements.lock
python starter/baseline.py --window 50 --out generated
```

The output is `generated/baseline_smooth50.csv`. The previously verified prepared output has SHA-256 `d444caa1aebb6fb58386033a5967c83302b8dd984ea91c86381bf208332702ff`. Compare hashes; differences in environment or line endings must be investigated. The separately supplied organizer validator can check the file structure. Do not publish generated CSVs or match data.

Two standalone Windows x86-64 / CPython 3.13.17 CLI runs from this package produced the recorded hash. The hash lock applies only to the specified Windows wheels; other platforms need separately verified hashes. Confirm organizer eligibility before submitting.
