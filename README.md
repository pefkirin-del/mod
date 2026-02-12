 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/README.md b/README.md
new file mode 100644
index 0000000000000000000000000000000000000000..27ab698b474ee070dc3f49de4d8c4fa6bb7fc067
--- /dev/null
+++ b/README.md
@@ -0,0 +1,14 @@
+# CULT Chess Project Docs
+
+This repository now keeps the primary product requirements document in:
+
+- `docs/PRODUCT_SPEC.md`
+
+If you are sharing requirements with developers, paste updates directly into that file so product, smart-contract, backend, and frontend work stay aligned.
+
+## Recommended next documents
+
+- `docs/ARCHITECTURE.md` — system design and component diagrams
+- `docs/CONTRACT_SPEC.md` — escrow and settlement contract details
+- `docs/API_SPEC.md` — backend endpoints and websocket events
+- `docs/TEST_PLAN.md` — acceptance and edge-case testing
 
EOF
)
