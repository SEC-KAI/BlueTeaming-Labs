# Email Header Analysis

## Analysis goal

The main question is not simply “Does this email look suspicious?” The goal is to determine whether the sender identity, mail path, authentication, content, and follow-on activity tell the same story.

## 1. Start with identity fields

For the spoofed-sender sample, the message presents `info@mutawamarine.com` as the sender. I compare the visible identity with the envelope sender, transport path, and authentication results instead of trusting the display name.

## 2. Reconstruct the route

The oldest useful `Received` line identifies `192.119.71.157` and `hwsrv-737338.hostwindsdns.com` as the source visible at the first external handoff in the captured chain. The HELO string claims `mutawamarine.com`, but HELO is only a claimed SMTP identity and is not proof by itself.

## 3. Compare authentication

The receiving system recorded SPF failure. The published SPF record ends in `-all` and delegates authorized sending to Microsoft's SPF infrastructure. The observed source did not satisfy that policy.

The evidence also records `dmarc=unknown`. I preserve that exact result rather than changing it to a failure. No DKIM result is visible in the selected header excerpt.

## 4. Analyze links separately from sender authentication

A phishing email can pass some authentication checks and still be malicious, and a legitimate email can contain an external link. For that reason, I also inspect the actual destination and look for:

- Lookalike spelling such as `m1crosoftsupport.co`
- Unrelated redirect infrastructure such as `kennaroads.buzz`
- Login or credential-collection behavior
- Phishing kits or archived web content
- User interaction visible in proxy/firewall/network logs

## 5. Correlate with downstream telemetry

The strongest suspicious-link example included an allowed HTTPS connection to `m1crosoftsupport.co/login`. That confirmed the endpoint reached the URL rather than merely receiving it in an email.

By contrast, the HR onboarding alert was closed false positive because the sender/domain matched approved business context and no suspicious link activity was identified.

## Conclusion

The final classification came from **multiple aligned indicators**, not one field:

- Sender authentication did not support the `mutawamarine.com` claim.
- Malicious-link exercises showed impersonation, suspicious infrastructure, phishing-kit behavior, and/or network interaction.
- The legitimate HR message had business context and no corroborating malicious activity.

This approach reduces both missed phishing and unnecessary escalation of legitimate email.
