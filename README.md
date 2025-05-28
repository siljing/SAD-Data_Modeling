# Data Modeling

### **1. 实体定义矩阵（Entity Definition Matrix）**  
| **实体名称**          | **业务定义**                                                                 |
|-----------------------|----------------------------------------------------------------------------|
| **Employee**          | 公司员工的核心信息，包括唯一标识、姓名、联系方式、雇佣状态等。每个员工拥有唯一的EmployeeID，即使离职后ID也不会重复使用。需支持员工自助更新个人信息（如地址、电话），并关联其所属部门、紧急联系人、捐款和储蓄债券记录。                                       | 
| **Department**        | 公司部门信息，记录部门编号、名称及所属站点（如Orlando、Denver）。每个部门属于一个站点，一名员工仅归属于一个部门。部门信息用于组织架构管理和数据权限控制。                                               |
| **EmergencyContact**  | 员工的紧急联系人信息，包括姓名、关系、联系方式（家庭电话、工作电话）。一名员工可登记多个紧急联系人（如配偶、父母）。                                           | 
| **UnitedWayContribution** | 员工对United Way的捐款记录。包括捐款类型（固定比例、一次性金额、自定义金额）、生效日期、扣款频率（每期工资扣除）。每位员工每年需提交一次更新表单。                            | 
| **SavingsBond**       | 员工购买的美国储蓄债券记录。每笔债券需记录面额（$100、$200等）、扣款类型（一次性或定期）、债券所有者信息（员工本人或其他受益人，如子女）。每位员工可为多个受益人购买债券，需填写独立表单。                              |
| **Site**              | 公司站点信息，包括站点名称（如Orlando、Denver）、地址、员工数量。用于管理多站点数据分布，确保电话簿和邮件投递按站点分类。                                    |
| **TelephoneListing**  | 员工电话及办公地点信息，包括分机号、建筑编号、房间号、邮件停靠点（Mail Stop）。与员工实体直接关联，需支持实时更新以替代纸质电话簿。                                        |
| **BondOwner**         | 储蓄债券的所有者信息（可能为非员工，如员工子女）。记录所有者的姓名、SSN、联系地址，确保债券归属清晰。                                   |
| **EmploymentHistory**         | （高级选项）员工的雇佣历史记录，包括入职日期、离职日期、雇佣状态（在职、退休、离职、休假）、雇佣类型（全职/兼职）。支持员工多次入职的场景，保留历史数据以计算福利资格。                                   |
---

#### 补充说明    
1. **关键业务规则**：  
   - **EmployeeID**为不可重复的序列号，即使员工离职也不回收。  
   - SavingsBond的所有者（**BondOwner**）需独立管理，避免与员工信息混淆。  
   - **TelephoneListing**的更新需实时同步至所有关联系统（如工资系统、邮件系统）。  

2. **高级扩展**：  
   - **EmploymentHistory**实体支持员工状态跟踪，通过`EmploymentStatus`字段（枚举值：Active/Retired/Terminated/Leave）和`EmploymentType`字段（Full-Time/Part-Time）实现灵活管理。  
   - 通过`HireDate`和`TerminationDate`记录多次雇佣历史，确保福利计算的准确性（如养老金、医疗保险）。  

---

### **2. 上下文数据模型（Context Data Model）**  
![](https://github.com/siljing/SAD-Data_Modeling/blob/main/Context%20Data%20Model.svg?text=Context+Data+Model+Diagram)  
#### **核心实体及其关系**  
1. **Employee**（员工）  
   - **关系**：  
     - **属于（belongs to）** → **Department**（部门）：一名员工属于一个部门（1:N）。  
     - **登记（registers）** → **EmergencyContact**（紧急联系人）：一名员工可登记多个紧急联系人（1:N）。  
     - **参与（participates in）** → **UnitedWayContribution**（United Way捐款）：一名员工可提交多次捐款记录（1:N）。  
     - **购买（purchases）** → **SavingsBond**（储蓄债券）：一名员工可购买多笔债券（1:N）。  
     - **关联（is listed in）** → **TelephoneListing**（电话簿列表）：每位员工对应一条电话簿记录（1:1）。  
     - **记录（has）** → **EmploymentHistory**（雇佣历史，高级选项）：一名员工可有多条雇佣状态记录（1:N）。  

2. **Department**（部门）  
   - **关系**：  
     - **位于（is located in）** → **Site**（站点）：一个站点包含多个部门（1:N）。  

3. **SavingsBond**（储蓄债券）  
   - **关系**：  
     - **归属（is owned by）** → **BondOwner**（债券所有者）：每笔债券对应一个所有者（1:1）。  

---

#### **关系说明**  
1. **Employee ↔ Department**  
   - **方向命名**：员工属于（belongs to）部门。 
   -  **基数**：`Employee (1..n) → Department (1..1)`  
   - **业务规则**：每个员工必须归属于一个部门，一个部门可包含多名员工。

2. **Employee ↔ EmergencyContact**  
   - **方向命名**：员工登记（registers）紧急联系人。  
   - **基数**：`Employee (1..1) → EmergencyContact (0..n)`  
   - **业务规则**：紧急联系人信息可选，但需确保至少一个有效联  
3. **Employee ↔ UnitedWayContribution**  
   - **方向命名**：员工参与（participates in）United Way捐款。  
   - **基数**：`Employee (1..1) → UnitedWayContribution (0..n)`  
   - **业务规则**：每年需更新捐款选项，支持一次性或周期性扣款。

4. **Employee ↔ SavingsBond**  
   - **方向命名**：员工购买（purchases）储蓄债券。  
   - **基数**：`Employee (1..1) → SavingsBond (0..n)`  
   - **业务规则**：每笔债券需独立记录所有者信息，支持为多个受益人购买。  

5. **SavingsBond ↔ BondOwner**  
   - **方向命名**：储蓄债券归属（is owned by）所有者。  
   -  **基数**：`SavingsBond (1..n) → BondOwner (1..1)`  
   - **业务规则**：所有者可为员工本人或第三方（如子女），需记录SSN和地址。  

6. **Department ↔ Site**  
   - **方向命名**：部门位于（is located in）站点。  
   - **基数**：`Department (1..n) → Site (1..1)`  
   - **业务规则**：所有部门必须关联到一个站点，支持多站点数据隔离。  

7. **Employee ↔ TelephoneListing**  
   - **方向命名**：员工关联（is listed in）电话簿记录。  
   - **基数**：`Employee (1..1) ↔ TelephoneListing (1..1)`  
   - **业务规则**：电话簿信息需实时更新，替代纸质版本。  

8. **Employee ↔ EmploymentHistory**（高级选项）  
   - **方向命名**：员工记录（has）雇佣历史。  
   - **基数**：`Employee (1..1) → EmploymentHistory (1..n)`  
   - **业务规则**：支持多次雇佣记录，区分状态（在职、退休、离职、休假）和类型（全职/兼职）。  

--- 
#### **基数总结表**  
| **关系**                  | **父实体基数** | **子实体基数** | **说明**                                      |  
|---------------------------|----------------|----------------|---------------------------------------------|  
| Employee → Department      | 1..1           | 1..n           | 员工必须属于一个部门，部门可包含多名员工。            |  
| Employee → EmergencyContact | 1..1           | 0..n           | 员工可选登记多个紧急联系人。                      |  
| Employee → UnitedWayContribution | 1..1       | 0..n           | 员工可选多次捐款。                             |  
| Employee → SavingsBond     | 1..1           | 0..n           | 员工可选购买多笔债券。                          |  
| Employee ↔ TelephoneListing | 1..1          | 1..1           | 员工与电话簿记录一一对应。                       |  
| Employee → EmploymentHistory | 1..1          | 1..n           | 员工至少有一条雇佣记录，可有多条历史。              |  
| Department → Site          | 1..n           | 1..1           | 部门必须属于一个站点，站点可包含多个部门。            |  
| SavingsBond → BondOwner    | 1..n           | 1..1           | 每笔债券对应一个所有者，所有者可拥有多笔债券。         |  

---

#### **关键点说明**  
1. **强制关系**：  
   - 如`Employee → Department`、`Employee ↔ TelephoneListing`，确保数据完整性（如每位员工必须归属部门）。  
2. **可选关系**：  
   - 如`Employee → EmergencyContact`、`Employee → SavingsBond`，允许灵活的业务场景（如员工可能不登记紧急联系人）。  
3. **高级选项扩展**：  
   - `EmploymentHistory`实体支持复杂雇佣场景（如多次离职/复职），通过基数`1..n`强制记录至少一条历史。  

### **3. 基于键的数据模型（Key-Based Data Model）**  
![](https://github.com/siljing/SAD-Data_Modeling/blob/main/Key-Based%20Data%20Model.svg?text=Key-Based+Data+Model+Diagram)  
---

#### **实体主键定义**  
| **实体名称**          | **主键**              | **说明**                                                                 |  
|-----------------------|-----------------------|------------------------------------------------------------------------|  
| **Employee**          | `EmployeeID`（序列码） | 唯一标识员工，即使员工离职也不重复使用。                                   |  
| **Department**        | `DepartmentID`（序列码） | 部门唯一标识符 |  
| **EmergencyContact**  | `EmployeeID`（引用Employee表）+`ContactSequence`（序号1或2） | 示例：员工ID为1001的主要联系人主键为(1001, 1)，次要联系人为(1001, 2)。                                               |  
| **UnitedWayContribution** | `ContributionID`（序列码） | 唯一标识每次捐款记录。                                               |  
| **SavingsBond**       | `BondID`（序列码）    |  唯一标识每笔债券。    |  
| **Site**              | `SiteCode`（字母编码） | 三位字母编码（如ORL、DEN）。                                           |  
| **TelephoneListing**  | `ListingID`（序列码）  |  唯一标识每条记录                          |  
| **BondOwner**         | `OwnerSSN`            | 使用所有者的社会安全号码（SSN）作为唯一标识。                            |  
| **EmploymentHistory** | `HistoryID`（序列码） | 唯一标识雇佣历史记录。                                                 |  
---
#### **关键点说明** 
1. **主键选择**：  
   - **稳定性**：使用序列码（如`EmployeeID`、`DepartmentID`）或代理键（如`ContributionID`），避免因业务规则变化导致主键失效。  
   - **唯一性**：确保每个实体的主键唯一标识实例（如`OwnerSSN`唯一标识债券所有者）。  
   - **业务编码**：部分实体使用字母编码（如`SiteID`为ORL、DEN），提升业务可读性。  
2. **债券所有者独立性**：  
   - BondOwner的主键`OwnerSSN`独立于Employee，支持非员工所有者（如员工子女）。


### **4. 完全属性化的数据模型（Fully Attributed Data Model）**  
![](https://github.com/siljing/SAD-Data_Modeling/blob/main/Fully%20Attributed%20Data%20Model.svg?text=Fully+Attributed+Data+Model+Diagram)  

#### **Employee（员工）实体**  
| **属性名称**          | **数据类型**       | **约束**                         | **说明**                                                                 |  
|-----------------------|--------------------|----------------------------------|-------------------------------------------------------------------------|  
| **EmployeeID**        | 整数（序列码）      | 主键（PK）                       | 员工唯一标识符，由系统自动生成，永不重复。                                       |  
| **OfficalHireDate**          | 日期               | 非空（NOT NULL）                 | 正式入职日期（非填表日期），即使员工延迟报到也记录原定入职日                                              |  
| **LastName**          | 字符串（50字符）    | 非空（NOT NULL）                 | 员工姓氏，需符合字符集规范（如仅允许字母、连字符）。                                 |  
| **FirstName**         | 字符串（50字符）    | 非空（NOT NULL）                 | 员工名字，需符合字符集规范。                                                   |  
| **MiddleName**     | 字符（30字符）       | 可为空（NULL）                   | 替换原MiddleInitial，记录完整中间名（如"Carl"而非仅"C"）                                                 |  
| **Nickname**     | 字符（30字符）       | 可为空（NULL）                   | 记录员工常用昵称（如"Johnny"对应正式名"John"）                                                    |
| **SSN**               | 字符串（11字符）    | 非空（NOT NULL）、唯一（UNIQUE） | 社会保险号，格式为`XXX-XX-XXXX`。                                           |  
| **HomePhone**         | 字符串（15字符）    | 非空（NOT NULL）                 | 家庭电话，格式为`(XXX) XXX-XXXX`。                                          |  
| **BirthDate**         | 日期               | 非空（NOT NULL）                 | 出生日期，格式为`YYYY-MM-DD`。                                              |  
| **HomeAddress**       | 字符串（200字符）   | 非空（NOT NULL）                 | 家庭地址，包含街道、城市、州、邮编（如`1456 Forest Drive, Orlando, FL 32859-0032`）。 |  
| **MaritalStatus**     | 枚举（M/S）        | 非空（NOT NULL）                 | 婚姻状况：`M`（已婚）、`S`（单身）。                                           |  

---

#### **EmergencyContact（紧急联系人）实体**  
| **属性名称**          | **数据类型**       | **约束**                         | **说明**                                                                 |  
|-----------------------|--------------------|----------------------------------|-------------------------------------------------------------------------|  
| **EmployeeID**        | 整数               | 外键（FK，引用Employee表）        | 关联员工的唯一标识符。                                                    |  
| **ContactSequence**   | 整数               | 主键（PK，联合键部分）            | 紧急联系人序号（如1表示主要联系人，2表示次要联系人）。                             |  
| **LastName**          | 字符串（50字符）    | 非空（NOT NULL）                 | 联系人姓氏，需符合字符集规范。                                               |  
| **FirstName**         | 字符串（50字符）    | 非空（NOT NULL）                 | 联系人名字，需符合字符集规范。                                               |  
| **MiddleName**     | 字符（1字符）       | 可为空（NULL）                   | 中间名                                                   |  
| **Relationship**      | 字符串（20字符）    | 非空（NOT NULL）                 | 与员工的关系（如“Spouse”“Father”）。                                        |  
| **HomeAddress**       | 字符串（200字符）   | 非空（NOT NULL）                 | 联系人的家庭地址，格式同员工地址。                                             |  
| **HomePhone**         | 字符串（15字符）    | 非空（NOT NULL）                 | 联系人的家庭电话，格式为`(XXX) XXX-XXXX`。                                   |  
| **WorkPhone**         | 字符串（15字符）    | 可为空（NULL）                   | 联系人的工作电话，格式为`(XXX) XXX-XXXX`或“Retired”等描述。                      |  

---

##### **键设计说明**  
1. **Employee实体主键**：  
   - **EmployeeID**：使用序列码（如`1001, 1002`），确保唯一性且不因业务规则变化失效。  
   - **SSN**：作为唯一约束（UNIQUE），但不作为主键，避免隐私泄露风险。  

2. **EmergencyContact实体主键**：  
   - 联合主键：**EmployeeID**（引用Employee表） + **ContactSequence**（序号）。  
   - 示例：员工ID为`1001`的主要联系人主键为`(1001, 1)`，次要联系人为`(1001, 2)`。  

3. **外键约束**：  
   - **EmployeeID**在EmergencyContact表中为非空外键，确保每个紧急联系人必须关联有效员工。  

---

##### **业务规则验证**  
1. **强制字段**：  
   - Employee表中的`HireDate`、`LastName`、`FirstName`、`SSN`、`HomePhone`、`BirthDate`、`HomeAddress`、`MaritalStatus`必须非空。  
   - EmergencyContact表中的`LastName`、`FirstName`、`Relationship`、`HomeAddress`、`HomePhone`必须非空。  

2. **数据格式**：  
   - **SSN**：强制格式`XXX-XX-XXXX`，通过正则表达式验证。  
   - **Phone**：强制格式`(XXX) XXX-XXXX`，允许扩展（如分机号）。  
   - **Date**：统一使用ISO标准格式`YYYY-MM-DD`。  

3. **可选字段**：  
   - **MiddleInitial**和**WorkPhone**可为空，符合业务场景需求（如无中间名或退休人员无工作电话）。  

---
#### **UnitedWayContribution（United Way捐款）实体**
| **属性名称**          | **数据类型**         | **约束**                         | **说明**                                                                 |  
|-----------------------|----------------------|----------------------------------|-------------------------------------------------------------------------|  
| **ContributionID**    | 整数（序列码）        | 主键（PK）                       | 捐款记录唯一标识符                                                       |  
| **EmployeeID**        | 整数                 | 外键（FK，引用Employee表）        | 关联员工的唯一标识符                                                     |  
| **ContributionYear**  | 整数（年份）          | 非空（NOT NULL）                 | 捐款生效年份（如2000）                                                   |  
| **DeductionType**     | 枚举                 | 非空（NOT NULL）                 | 捐款类型：<br>`FAIR_SHARE`（5%工资）<br>`ONE_TIME_GIFT`（一次性）<br>`OTHER_AMOUNT`（自定义）<br>`NO_CONTRIBUTION`（不捐） |  
| **DeductionAmount**   | 小数（10,2）         | 条件非空                         | 仅当类型为`ONE_TIME_GIFT`或`OTHER_AMOUNT`时必填（如25.00）                |  
| **EffectiveDate**     | 日期                 | 非空（NOT NULL）                 | 生效日期（新员工为入职日，老员工为次年1月1日）                              |  
| **Signature**         | 字符串（100字符）     | 非空（NOT NULL）                 | 员工签名（电子签名或物理签名扫描）                                        |  
| **SignatureDate**     | 日期                 | 非空（NOT NULL）                 | 签名日期                                                                 |  

---

##### **关键业务规则与设计说明**
1. **预填信息关联**  
   - 表单预填的`EmployeeID`、`ManagerID`、`DepartmentID`通过外键关联其他实体，避免数据冗余
2. **生效日期规则**  
   - 新员工：`EffectiveDate` = `OfficialHireDate`（正式入职日）
   - 老员工：`EffectiveDate` = `ContributionYear-01-01`（如2000-01-01）

3. **唯一性约束**  
   - 联合唯一键：`EmployeeID` + `ContributionYear`，确保每年仅一条有效记录

---
#### **SavingsBond（储蓄债券）实体**
| **属性名称**          | **数据类型**         | **约束**                         | **说明**                                                                 |  
|-----------------------|----------------------|----------------------------------|-------------------------------------------------------------------------|  
| **BondID**            | 整数（序列码）        | 主键（PK）                       | 债券唯一标识符                                                          |  
| **EmployeeID**        | 整数                 | 外键（FK，引用Employee表）        | 购买债券的员工ID                                                        |  
| **OwnerSSN**        | 整数                 | 外键（FK，引用BondOwener表）        | 债券所有者的SSN                                                       |  
| **Denomination**      | 枚举                 | 非空（NOT NULL）                 | 债券面额：<br>`100`/`200`/`500`/`1000`（美元）                              |  
| **DeductionType**     | 枚举                 | 非空（NOT NULL）                 | 扣款类型：<br>`ONE_TIME`（一次性）<br>`REGULAR`（定期）                     |  
| **DeductionAmount**   | 小数（10,2）         | 非空（NOT NULL）                 | 扣款金额（根据面额自动计算或手动输入）                                       |   
| **OwnType**   | 字符串（30字符）      | 非空（NOT NULL）                   | CoOwner或     Beneficiary                                                   |                                       |  
| **MailingAddress**    | 字符串（200字符）     | 非空（NOT NULL）                 | 债券邮寄地址（同员工地址或自定义）                                                                             |    
| **Signature**         | 字符串（100字符）     | 非空（NOT NULL）                 | 员工电子签名                                                            |  
| **SignatureDate**     | 日期                 | 非空（NOT NULL）                 | 签名日期                                                                |  
                              

---
#### **BondOwner（债券所有者）实体**
| **属性名称**          | **数据类型**         | **约束**                         | **说明**                                                                 |  
|-----------------------|----------------------|----------------------------------|-------------------------------------------------------------------------|   
| **SSN**               | 字符串（11字符）      | 主键（PK） | 所有者SSN（格式`XXX-XX-XXXX`）                                          |  
| **LastName**          | 字符串（50字符）      | 非空（NOT NULL）                 | 所有者姓氏                                                               |  
| **FirstName**         | 字符串（50字符）      | 非空（NOT NULL）                 | 所有者名字                                                               |  
| **MiddleName**        | 字符串（30字符）      | 可为空（NULL）                   | 所有者中间名                                                             | 
---
##### **关键业务规则与设计说明**
1. **独立所有者管理**  
   - 相同所有者（如员工子女）购买多笔债券时，只需在`BondOwner`中存储一次信息
--- 
#### **TelephoneListing（电话簿列表）实体**
| **属性名称**          | **数据类型**         | **约束**                         | **说明**                                                                 |  
|-----------------------|----------------------|----------------------------------|-------------------------------------------------------------------------|  
| **ListingID**         | 整数（序列码）        | 主键（PK）                       | 记录唯一标识符                                                          |  
| **EmployeeID**        | 整数                 | 外键（FK，引用Employee表）        | 关联员工的唯一标识符（非空）                                              |  
| **PhoneNumber**       | 字符串（15字符）      | 非空（NOT NULL）                 | 工作电话（格式`(XXX) XXX-XXXX`）                                         |  
| **SiteCode**          | 字符串（5字符）       | 非空（NOT NULL）                 | 站点代码（如ORL/ DEN/ MAR等），外键引用Site表                              |  
| **BuildingCode**      | 字符串（10字符）      | 非空（NOT NULL）                 | 建筑编号（如E7/ X.1/ AA等）                                              |  
| **RoomNumber**        | 字符串（10字符）      | 非空（NOT NULL）                 | 房间号（如2234/ 12/ 560等）                                              |  
| **MailStop**          | 字符串（10字符）      | 非空（NOT NULL）                 | 邮件停靠点（如359/ A12/ 5512等），Dotty解释为"公司内部邮编"                  |  
| **DepartmentID**      | 整数                 | 外键（FK，引用Department表）      | 员工所属部门（来自表单的Dept列）                                          |  

---
#### **EmploymentHistory（雇佣历史）**  
| **属性名称**          | **数据类型**         | **约束**                         | **说明**                                                                 |  
|-----------------------|----------------------|----------------------------------|-------------------------------------------------------------------------|  
| **HistoryID**         | 整数（序列码）        | 主键（PK）                       | 雇佣记录唯一标识符                                                       |  
| **EmployeeID**        | 整数                 | 外键（FK，引用Employee表）        | **永不重用**（Dotty强调即使员工离职ID也保留）                               |  
| **EmploymentType**    | 枚举                 | 非空（NOT NULL）                 | 雇佣类型：<br>`FULL_TIME`（全职）/`PART_TIME`（兼职）                     |  
| **EmploymentStatus**  | 枚举                 | 非空（NOT NULL）                 | 雇佣状态：<br>`ACTIVE`（在职）/`RETIRED`（退休）/`TERMINATED`（离职）<br>`LEAVE`（休假）/`INACTIVE`（未激活） |  
| **StartDate**         | 日期                 | 非空（NOT NULL）                 | 状态生效开始日期                                                         |  
| **EndDate**           | 日期                 | 可为空（NULL）                   | 状态结束日期（如离职日，NULL表示当前状态）                                 |  
| **TerminationReason** | 字符串（50字符）      | 可为空（NULL）                   | 离职原因（仅当状态为`TERMINATED`时必填）                               |  
---
