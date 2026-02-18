# Valenfind - TryHackMe CTF

Room: https://tryhackme.com/room/lafb2026e10

---

## 🇵🇱 Opis

Ten write-up przedstawia rozwiązanie wyzwania Valenfind na platformie TryHackMe.  
Podczas rozwiązywania zadania wykorzystano podatność typu LFI (Local File Inclusion), 
uzyskano dostęp do kodu źródłowego aplikacji, wyodrębniono klucz API administratora, 
a następnie wyeksportowano bazę danych w celu zdobycia flagi.

Wyzwanie pokazuje, jak pozornie niewielka podatność może prowadzić do pełnego kompromisu aplikacji.

---

## 🇬🇧 Description

This write-up explains how the Valenfind TryHackMe challenge was solved by exploiting a Local File Inclusion (LFI) vulnerability, accessing the application source code, extracting an admin API key, and exporting the database to retrieve the final flag.

The challenge demonstrates how a seemingly small vulnerability can lead to full application compromise.

---
