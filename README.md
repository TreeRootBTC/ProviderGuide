# Accessing the TreeRoot Provider API in Frontend Projects

This guide will walk you through how to access and use the TreeRoot Provider API in your frontend projects. This will allow your web applications to interact with the TreeRoot wallet extension.

## Basic Provider Access

The TreeRoot Provider is injected into web pages as a global object called `treeroot`. Here's how to check if the provider is available:

```javascript
if (window.treeroot) {
  // TreeRoot provider is available
  console.log("TreeRoot wallet is installed");
} else {
  // TreeRoot provider is not available
  console.log("Please install TreeRoot wallet extension");
}
```

## Core API Methods

### 1. Connecting to the Wallet

To request a connection to the wallet:

```javascript
async function connectWallet() {
  try {
    // This will prompt the user with a connection request
    const accounts = await window.treeroot.requestConnection();
    
    if (accounts && accounts.length > 0) {
      console.log("Connected account:", accounts[0]);
      // Store the account address for later use
      const currentAccount = accounts[0];
    }
  } catch (error) {
    console.error("Connection error:", error.message);
  }
}
```

### 2. Getting Connected Accounts

To check if the user is already connected and get their accounts:

```javascript
async function getAccounts() {
  try {
    const accounts = await window.treeroot.getAccounts();
    
    if (accounts && accounts.length > 0) {
      console.log("Already connected accounts:", accounts);
      return accounts;
    } else {
      console.log("No connected accounts");
      return [];
    }
  } catch (error) {
    console.error("Error getting accounts:", error.message);
    return [];
  }
}
```

### 3. Signing Messages

To request the user to sign a message:

```javascript
async function signMessage(message) {
  try {
    // This will prompt the user with a signature request
    const signature = await window.treeroot.signMessage(message);
    console.log("Signature:", signature);
    return signature;
  } catch (error) {
    console.error("Signature error:", error.message);
    throw error;
  }
}
```

## Listening for Events

The TreeRoot provider emits events when accounts change:

```javascript
// Listen for account changes
window.addEventListener('treeroot_accountsChanged', (event) => {
  const accounts = event.detail;
  console.log("Accounts changed:", accounts);
  
  if (accounts.length > 0) {
    // Update your UI with the new account
    updateUI(accounts[0]);
  } else {
    // User disconnected their wallet
    handleDisconnect();
  }
});
```

## Complete Integration Example

Here's a complete example of how to integrate the TreeRoot provider in a React application:

```jsx
import { useState, useEffect } from 'react';

function TreeRootWallet() {
  const [account, setAccount] = useState(null);
  const [isConnected, setIsConnected] = useState(false);
  const [message, setMessage] = useState('Hello from TreeRoot!');
  const [signature, setSignature] = useState('');
  const [error, setError] = useState('');

  // Check if provider exists
  useEffect(() => {
    if (!window.treeroot) {
      setError('TreeRoot wallet extension not installed');
    } else {
      // Check if already connected
      checkConnection();
      
      // Listen for account changes
      window.addEventListener('treeroot_accountsChanged', handleAccountsChanged);
      
      return () => {
        window.removeEventListener('treeroot_accountsChanged', handleAccountsChanged);
      };
    }
  }, []);
  
  // Handle account changes
  const handleAccountsChanged = (event) => {
    const accounts = event.detail;
    
    if (accounts.length > 0) {
      setAccount(accounts[0]);
      setIsConnected(true);
    } else {
      setAccount(null);
      setIsConnected(false);
      setSignature('');
    }
  };
  
  // Check if already connected
  const checkConnection = async () => {
    try {
      const accounts = await window.treeroot.getAccounts();
      
      if (accounts && accounts.length > 0) {
        setAccount(accounts[0]);
        setIsConnected(true);
      }
    } catch (err) {
      console.error('Error checking connection:', err);
    }
  };
  
  // Connect wallet
  const connectWallet = async () => {
    try {
      setError('');
      const accounts = await window.treeroot.requestConnection();
      
      if (accounts && accounts.length > 0) {
        setAccount(accounts[0]);
        setIsConnected(true);
      }
    } catch (err) {
      setError(err.message);
    }
  };
  
  // Sign message
  const signMessage = async () => {
    try {
      setError('');
      if (!message) {
        setError('Please enter a message to sign');
        return;
      }
      
      const sig = await window.treeroot.signMessage(message);
      setSignature(sig);
    } catch (err) {
      setError(err.message);
    }
  };
  
  return (
    <div className="wallet-container">
      <h2>TreeRoot Wallet Integration</h2>
      
      {error && (
        <div className="error-message">
          {error}
        </div>
      )}
      
      <div className="connection-status">
        Status: {isConnected ? 'Connected' : 'Disconnected'}
      </div>
      
      {account && (
        <div className="account-info">
          Account: {account}
        </div>
      )}
      
      {!isConnected ? (
        <button onClick={connectWallet}>
          Connect to TreeRoot
        </button>
      ) : (
        <div className="signing-section">
          <input
            type="text"
            value={message}
            onChange={(e) => setMessage(e.target.value)}
            placeholder="Enter message to sign"
          />
          <button onClick={signMessage}>
            Sign Message
          </button>
          
          {signature && (
            <div className="signature-result">
              <h3>Signature:</h3>
              <pre>{signature}</pre>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

export default TreeRootWallet;
```

## Best Practices

1. **Always check if the provider exists** before trying to use it
2. **Handle errors gracefully** - provide clear feedback to users
3. **Listen for account changes** to keep your UI in sync
4. **Don't store sensitive data** in localStorage or sessionStorage
5. **Implement proper loading states** during async operations
6. **Test thoroughly** with the actual extension installed

## API Reference

Here's a quick reference of all available methods in the TreeRoot provider:

| Method | Description | Parameters | Return Value |
|--------|-------------|------------|--------------|
| `getAccounts()` | Get currently connected accounts | None | Promise<string[]> |
| `requestConnection()` | Request connection to the wallet | None | Promise<string[]> |
| `signMessage(message)` | Request signature for a message | message: string | Promise<string> |

## Events

| Event Name | Description | Event Detail |
|------------|-------------|--------------|
| `treeroot_accountsChanged` | Fired when accounts change | string[] (array of account addresses) |

## Testing Your Integration

You can use the test dapp included in the TreeRoot wallet repository to verify your integration:

1. Run the test server: `node serve-test-dapp.js`
2. Open `http://localhost:3000` in your browser
3. Make sure the TreeRoot wallet extension is installed and running
4. Test the connection and signature functionality

## Troubleshooting

If you encounter issues with the TreeRoot provider:

1. Make sure the extension is properly installed and enabled
2. Check the browser console for any error messages
3. Verify that you're accessing the provider correctly (`window.treeroot`)
4. Ensure your site is not blacklisted in the extension settings
5. Try refreshing the page after the extension is loaded

For additional help, refer to the TreeRoot wallet documentation or contact the development team.
