![Use Case Diagram](https://uml.planttext.com/plantuml/png/dLJ9Rjim4BtpAmO-DGVd0u94Y66dXL4d2AesfmB1X1vR0rEaGL9DrUNVEv9CXKhH5kX9q5k6Cu_BTOZeOrCRPMWN8JhmlAyYRDcJZkoi2WUsH_HOmn7r0VTKnjOGT3JQlI6bK4VdORRmkk8l7E8CCC2dG3unxvt5cc4G42pkrxr4ld1vUVCrahTee6n3fBeeb1fhm-psNGaRvmbM6F490vraC_Yk09f0kiDc9Q4KXsKbZMgf1oiN0fCVMba-3dpkUCkcqpsrK5ey46pOaWoN30FTKxdhWUaPxj1xZDqa7xac8yRaIl-vPMt3KUR9UhltRiTcD9q4ufbCr5fTUMVv6y47CjI1jnHnApcCKuccmoOTakTT2qkqpa8PCJOXoGUgPrujze26fJ0DDjQo6pSsScnXlHhT9yrwDOdHu1xkd9qF2owTtHdMSOnHw8JbjetRGzSzB9ejnsuntoQFS3sgVYertGcODyVvZoBRKDKTCvZFSoo7nWt9mHIuwWvWygOeCopFIxtuuFoyc0O25tAEsMhJR4dERrRkVxHf_j-LMKF_4AjNsITvogENazmykoqA-6pVQ6UbYjpt2tYkIEuRXqVTBtHr1JiqWSx-L3FlwYzbXMsE8NgIbwJcK6FKrPbIrwALv-SN)

```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

actor "Archivist" as User
actor "Konami Archive API" as API <<External System>>

rectangle "MGS Lore Database System" {
  ' Базові прецеденти
  usecase "Search Character Dossier" as UC_Search
  usecase "Filter by Snake Lineage" as UC_FilterSnake
  usecase "View Narrative Timeline" as UC_Timeline
  usecase "Inspect Character Profile" as UC_Inspect
  
  ' Прецеденти синхронізації
  usecase "Synchronize Release Metadata" as UC_SyncMeta
  usecase "Verify Canon Status" as UC_VerifyCanon
  
  ' Допоміжні / розширені прецеденти
  usecase "Validate Canonical Entity ID" as UC_ValidateID
  usecase "Flag Non-Canon Conflict" as UC_FlagConflict
  usecase "Log Sync Audit Entry" as UC_LogAudit
}

' Generalization (Узагальнення)
UC_FilterSnake -up-|> UC_Search

' Взаємодія користувача
User --> UC_Search
User --> UC_Timeline
User --> UC_Inspect

' Взаємодія зовнішнього API
API --> UC_SyncMeta
API --> UC_VerifyCanon

' Зв'язки <<include>>
UC_Inspect ..> UC_ValidateID : <<include>>
UC_SyncMeta ..> UC_ValidateID : <<include>>
UC_SyncMeta ..> UC_LogAudit : <<include>>
UC_VerifyCanon ..> UC_LogAudit : <<include>>

' Зв'язки <<extend>>
UC_FlagConflict ..> UC_Timeline : <<extend>> \n(condition: where is_canon == false)
UC_FlagConflict ..> UC_VerifyCanon : <<extend>> \n(condition: on upstream mismatch)

@enduml

```