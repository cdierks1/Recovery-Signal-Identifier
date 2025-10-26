# Environment Setup for Stock Recovery Signal Identifier v10.6

## .env Auto-Loading Feature

The application now automatically checks for a `.env` file in the current directory on startup and can auto-populate configuration values.

### How to Use

1. Create a `.env` file in the same directory as the HTML file
2. Add the following variables as needed:

```bash
# Börsdata API key - get this from https://borsdata.se/
API_KEY=your_borsdata_api_key_here

# Path to instruments JSON file (relative to HTML file location)
INSTRUMENT_FILE=nordic_stocks.json
```

### What Happens Automatically

- **API_KEY**: If found in `.env`, it will auto-populate the API key input field
- **INSTRUMENT_FILE**: If found in `.env`, it will automatically load the specified instrument file and populate the instruments list

### User Feedback

The application provides visual feedback about the `.env` loading process:
- Status messages appear below the instrument file input
- Shows when `.env` file is loaded successfully
- Indicates when API key is set and instruments are loaded
- Reports any errors or failures clearly

### Error Handling

The application handles these scenarios gracefully:
- `.env` file doesn't exist (shows "file not found" message, continues normally)
- `.env` file exists but variable not found (uses default behavior)
- Invalid `.env` format (logs error but continues)
- Instrument file specified in `.env` can't be loaded (shows error in status)
- Network/CORS issues (shows "browser security" message, continues)

### Browser Security Note

Due to browser security restrictions, this feature **requires** a web server:
- ❌ **Does NOT work** when opened directly as a file (`file://` protocol)
- ✅ **Works** when served by a local web server (http:// protocol)

#### Solution 1: Use a Local Web Server
Install and use one of these simple web servers:
- **Python** (if installed): `python -m http.server 8000`
- **Node.js** (if installed): `npx http-server -p 8000`
- **PHP** (if installed): `php -S localhost:8000`

Then open: `http://localhost:8000/Stock%20Recovery%20Signal%20Identifier_v_10.6.html`

#### Solution 2: Manual Setup
If you don't want to use a web server, you can:
1. Manually enter your API key in the input field
2. Use the file picker to load your instrument file
3. The app will still work normally, just without auto-loading

### Example .env File

```bash
# Sample .env file
API_KEY=abcd1234efgh5678
INSTRUMENT_FILE=nordic_stocks.json
```
