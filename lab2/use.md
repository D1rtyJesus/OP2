![Use Case Diagram](https://uml.planttext.com/plantuml/png/dLNDJjj04BxlKuno0GxX0Q8GWHAAeW95JTjJ9RGuasI5iuvsDwMchTGVbVPGYLQguDX3No2YK50BuHNMRzHPE-iw0NhemR8zyytEV3FVsClQS6M6_P25s35W8b2YspFG5We38oB9zAwG0wvu7mOysELTR9auH71kBhiXCiO34oaehQYW9vu9RKh0DJpMgBpdVYHvNq06G5ZPQgOGkaErklhSe98yX5Qi3VPhDSRoiw6qiTQ2zKWXDBZXErpZ15Q25mnW3kmNExBdTcnFaYEmrybhUvOSs3DxIjTLyjPU46geCN2HfHPoAW7gFQ8JK5PeH5eBL5d1zUtCNuou9qA7sucX9VakmhgGI2tm0PaxTHMZdWZSWqsk53UExoFHnv02VPH_BuOqfHuGwq9jMohgY30FcW0e9kFzxIPLIDxO2tfyJwuntO-eCQFa83bAZe7y8x0drApVP3_rzgdkn3BegKYAVOI76A8pRg3XRUfytY72E5jMXWybnnDKeXFNkOoaWvQoTyWCBSFDKBDocI4DMsgI5GmY8Oski0SonMm8tUScw9LOHkyhCRYcwvAg_Ucut3hSZ5PAzppvQ2ySmbwHxt1g73mKH0ApQaH0MbkLHfWOcWrFp6EQZQdnXxmBcv4iPppgaUo48Z3v-CdjRSMmzQWBhbEmCcmBuvAfsCUGBxMoLupDmHfAL9HwdxjTWtdxtStC_aZdz8ceJEWii6crGNauAB-i5KHBXzdF982n6wmVRmNofMHk8Q5SBiOKJRaeYqOlkZiElxdNpEsqJ-ArKh0LP98UUcnFvf93uloBXbcj2Xc4mpRID-1lSbXSh4rF22gpK9_gVx1-83UHHKt_6tmBbLJfxLeDt1ewdOwJ3-GQsKidsriKEz7oEmACqqKb89iHMSwlK5cbaEofd4_tgW8pQxLmrqb-xBF7194acgJ92kpraBw_Gcy7QRwb9UZmKEC2Oyk4fb_77m00)

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
  
  ' Прецеденти синхронізації та валідації
  usecase "Synchronize Release Metadata" as UC_SyncMeta
  
  usecase UC_VerifyCanon as "Verify Canon Status
--
extension points:
OnCanonMismatch"

  ' Допоміжні прецеденти та розширення
  usecase "Validate Canonical Entity ID" as UC_ValidateID
  usecase "Flag Non-Canon Conflict" as UC_FlagConflict
  usecase "Log Sync Audit Entry" as UC_LogAudit
}

' Generalization (Узагальнення)
UC_FilterSnake -up-|> UC_Search

' Асоціації: Archivist
User --> UC_Search
User --> UC_Timeline
User --> UC_Inspect

' Асоціації: Konami Archive API
API --> UC_SyncMeta
API --> UC_VerifyCanon

' Зв'язки <<include>>
UC_Inspect ..> UC_ValidateID : <<include>>
UC_SyncMeta ..> UC_ValidateID : <<include>>
UC_SyncMeta ..> UC_LogAudit : <<include>>
UC_VerifyCanon ..> UC_LogAudit : <<include>>

' Зв'язки <<extend>> із точками розширення та умовами
UC_FlagConflict ..> UC_VerifyCanon : <<extend>>\n(point: OnCanonMismatch)
UC_FlagConflict ..> UC_Timeline : <<extend>>\n(condition: where is_canon == false)

@enduml

```