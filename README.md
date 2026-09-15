# insta_login

import { useState, useCallback } from "react";
import { View, Text, ActivityIndicator, StyleSheet } from "react-native";
import { WebView, type WebViewNavigation } from "react-native-webview";
import { CookieManager } from "expo-cookies";

const LOGIN_URL = "https://www.instagram.com/accounts/login";
const TARGET_COOKIE = "sessionid";
const COOKIE_DOMAIN = "https://www.instagram.com";
const LOGIN_PATH_FRAGMENT = "/login";
const DOMAIN_FRAGMENT = "instagram.com";

export default function LoginScreen() {
  const [isLoading, setIsLoading] = useState(true);
  const [sessionId, setSessionId] = useState<string | null>(null);

  const handleNavigationStateChange = useCallback(
    async (navState: WebViewNavigation) => {
      if (
        navState.url.includes(DOMAIN_FRAGMENT) &&
        !navState.url.includes(LOGIN_PATH_FRAGMENT) &&
        !sessionId
      ) {
        try {
          const cookies = await CookieManager.get(COOKIE_DOMAIN);
          const value = cookies[TARGET_COOKIE]?.value ?? null;

          if (value) {
            setSessionId(value);
            console.log("Session ID:", value);
          } else {
            console.log(`${TARGET_COOKIE} not found in cookie store`);
          }
        } catch (err) {
          console.error("Failed to read cookies:", err);
        }
      }
    },
    [sessionId]
  );

  const handleLoadEnd = () => setIsLoading(false);

  return (
    <View style={styles.container}>
      <WebView
        source={{ uri: LOGIN_URL }}
        onNavigationStateChange={handleNavigationStateChange}
        onLoadEnd={handleLoadEnd}
        incognito={false}
        thirdPartyCookiesEnabled
        sharedCookiesEnabled
        javaScriptEnabled
        domStorageEnabled
      />
      {isLoading && (
        <View style={styles.overlay}>
          <ActivityIndicator size="large" color="#208AEF" />
        </View>
      )}
      {sessionId && (
        <View style={styles.overlay}>
          <Text style={styles.statusText}>
            Session saved: {sessionId.slice(0, 12)}...
          </Text>
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  overlay: {
    position: "absolute",
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    alignItems: "center",
    justifyContent: "center",
    backgroundColor: "rgba(255,255,255,0.85)",
  },
  statusText: {
    marginTop: 12,
    fontSize: 16,
    color: "#208AEF",
  },
});
