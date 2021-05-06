

# 数字菜单

### 新增菜单

* api

> api/v1/pos/keyboard?stringified=true

```json
{"name":"沈测试","code":"017","type":"MAIN","row":"6","col":"6","pageable":true,"key_width":1,"key_height":1,"key_bg_color":{"PRODUCT":"#ffffff","CATEGORY":"#ffffff","EMPTY":"#ffffff","ACTION":"#ffffff","ALERT":"#ffffff"},"key_font_color":{"PRODUCT":"#939393","CATEGORY":"#939393","EMPTY":"#939393","ACTION":"#939393","ALERT":"#939393"}}
```

* 在metadata中创建一条keyboard，版本号为1

### 新增菜单明细

* api

> api/v1/pos/keyboard/4446187662608465920/keys?auto_build_keys=true&stringified=true

```json
{
    "items":[
        {
            "x":0,
            "y":0,
            "width":1,
            "high":1,
            "type":"CATEGORY",
            "bg_color":"",
            "font_color":"",
            "status":"Y",
            "is_overflow":false,
            "name":"茶叶-过时",
            "code":"001",
            "object_id":"4219461197910507521",
            "alias":""
        }
    ],
    "section_order":1
}
```

* 循环items，生成poskey的model

* 在主档中查询主表信息,校验信息

* 从item中查找出属于商品种类的数据，存到字段作为key

  ```json
  {
    '4219461198292189185': {'products': [], 'category_children_ids': []}, 
   '4219461197910507521': {'products': [], 'category_children_ids': []}
  }
  ```

  

### 查询菜单详情

> api/v1/pos/keyboard/4446187662608465920?relation=all&is_new=true&include_state=true&include_pending_record=true&stringified=true

### 菜单明细

> api/v1/pos/keyboard/4446187662608465920/keys?section_order=1&include_total_section=true&stringified=true

# 订货收货流程(跳转)

## 门店紧急订货

* 选择门店，新增紧急订单，提交审核，生成订单号， 342101210099，分配配送中心：广州黄埔仓(猩米科技）

* 仓库核验，执行通过，生成要货单号，

* 打开订单查询菜单，可通过订货单号，查询到要货单，记录JDE单号(21000574)，或者要货单号(122101210008)

* 打开收货--配送收货，根据要货单号，或者JDE，搜寻到收货单(412101210008)，此时应该是已出库状态

* 打开明细，填写数量，保存提交

* api执行顺序：

  1. http://bohtest.heytea.com/api/v2/supply/receiving/4449817476900880384/confirm

  ```json
  {
      "confirmed_products":[
          {
              "id":"4449817476980572160",
              "product_id":"4437200540039938048",
              "received_quantity":20,
              "confirmed_quantity":"10.000",
              "confirmed_accounting_quantity":"10.000"
          },
          {
              "id":"4449817477135761408",
              "product_id":"4437200540832661504",
              "received_quantity":25,
              "confirmed_quantity":"15.000",
              "confirmed_accounting_quantity":"15.000"
          }
      ],
      "lan":"zh-CN"
  }
  返回ture或者false
  ```

  2. http://bohtest.heytea.com/api/v2/supply/receiving/4449817476900880384?stringified=true&lan=zh-CN

     重新查询单据

  3. http://bohtest.heytea.com/api/v2/supply/receiving/4449817476900880384/product?stringified=true&lan=zh-CN

     查询单据明细

# 单据增加门店第三层管理区域展示

## 修改表结构

```sql
alter table metadata_store add column branch_region_relation LONGTEXT not NULL;
```

## 更改的proto

```
// 第三层管理区域
string third_branch_region = 33;
```

* supply
  * receiving_diff_bi:配送收货差异， 单独取的仓库主档，(两个完成)测试完毕
  * receiving_bi: 收货单， 单独取的仓库主档，(两个完成),测试完毕
  * supply:退货单报表, 更改了sql，join了仓库主档，测试完毕
  * adjust: GetAdjustBiCollect,报废单报表,更改了sql，join了仓库主档, （需要改两个接口), 测试完毕

* report
  * Opretion_report：GetMaterialSupplies

## 触发写入任务

```shell
curl --location --request POST 'http://127.0.0.1:32446/pub?topic=supply.demand.batch.init' \
--header 'Authorization: Bearer B8uVlLdvOEmhtce1SZoz1Q' \
--header 'Content-Type: application/json' \
--data-raw '{"partner_id":"2","user_id":"4201196624908648450", "mode":"create"}'
```



# Seesaw订单明细查询增加字段

## go环境搭建

* 创建目录 xxx
* go mod init xxx =>>>>创建go.mod
* 创建main.go，如果其中有依赖，运行时会自动下载依赖，并将依赖写到go.mod,其实会将依赖下载到GOPATH下的pkg的mod文件夹下(但是这样无法避免多个项目使用不同版本的问题), go还会自动生成一个 go.sum 文件来记录 dependency tree
* go mod vendor:将包从mod中转移到项目下的vendor目录下

## 修改

* 在pkg/utils/下新增了包
* 更改了proto：grpc/proto/indent.proto
* 更改了grpc/format.go

## 需求：增加公司编号，公司名称

* company_id
* Company_code
* Company_name

## config.yaml

```go
# Auto generate code. Thanks ['tianxuxin@126.com']
logger:
    level: info
    env: dev
grpc:
    port: 5001
db:
    dialect: mysql
    #connstr: xicha_stage:hex_xicha_ds123@tcp(rm-uf606j5022erh94it.mysql.rds.aliyuncs.com:3306)/supply_xicha_stage?parseTime=true&charset=utf8
    connstr: seesaw_boh_test:seesaw0617@tcp(rm-uf6xuw7c9aqu09c26so.mysql.rds.aliyuncs.com:3306)/seesaw_boh_test_receipt?charset=utf8&parseTime=true
    coststr: xicha_stage:hex_xicha_ds123@tcp(rm-uf606j5022erh94it.mysql.rds.aliyuncs.com:3306)/cost_xicha_stage?parseTime=True&loc=Local
    maxopen: 50
nsq:
    address: localhost:4150
    autoensuretopic: supplier_platform_auto_ensure
uuid:
    host: uuid
    port: 1234
flag:
    isplatform: true
externalpath:
    delivery: localhost:9089
    receive: localhost:8686
    review: localhost:9091
    metadata: localhost:5012
    oauth: localhost:9002
```

## Grpc/oauth

```go
package oauth

import (
	"context"
	fmt "fmt"
	math "math"

	grpc "google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
	"hexcloud.cn/histore/supplier-platform/config"
	"hexcloud.cn/histore/supplier-platform/pkg/logger"
	"hexcloud.cn/histore/supplier-platform/utils"
)

var (
	DefaultClient HexOauthClient
	Cache         map[uint64]string = make(map[uint64]string)
)

func Init() {
	conn, err := grpc.Dial(config.DefaultConfig.ExternalPath.Oauth, grpc.WithInsecure(), grpc.WithDefaultCallOptions(grpc.MaxCallRecvMsgSize(math.MaxInt32), grpc.MaxCallSendMsgSize(math.MaxInt32)))
	if err != nil {
		fmt.Println("Init oauth conn faild.")
		panic(err)
	}

	DefaultClient = NewHexOauthClient(conn)
}

func GetUserName(ctx context.Context, id uint64) string {
	if v, ok := Cache[id]; ok {
		return v
	}

	u, p, s := utils.GetUidAndPidString(ctx)
	fresh := metadata.AppendToOutgoingContext(context.Background(), "user_id", u, "partner_id", p, "scope_id", s)
	user, err := DefaultClient.GetUserById(fresh, &GetUserRequest{Id: id})
	if err != nil {
		logger.Pre().Error("Get user info error.", err)
		Cache[id] = "Unknown"
		user.Name  = "Unknown"
	} else {
		Cache[id] = user.Name
	}
	return user.Name
}

```

## format

```go
package grpc

import (
	"context"
	"fmt"
	"strconv"
	"time"

	"github.com/golang/protobuf/ptypes"
	"github.com/golang/protobuf/ptypes/timestamp"
	"github.com/spf13/cast"
	"google.golang.org/grpc/metadata"
	pb "hexcloud.cn/histore/supplier-platform/grpc/proto"
	entity "hexcloud.cn/histore/supplier-platform/grpc/proto/metadata"
	"hexcloud.cn/histore/supplier-platform/model"
	utils "hexcloud.cn/histore/supplier-platform/pkg/utils"
)

// func ModelOrdersToProtoOrders(orders []*model.OrderWithMoney) (result []*pb.OrderWithMoney) {
// 	for _, item := range orders {
// 		result = append(result, ModelOrderToProtoOrder(item))
// 	}
// 	return
// }

// func ModelOrderToProtoOrder(order *model.OrderWithMoney) *pb.OrderWithMoney {
// 	return &pb.OrderWithMoney{
// 		Id:               order.Id,
// 		Code:             order.Code,
// 		StoreSecondaryId: order.StoreSecondaryId,
// 		StoreName:        order.StoreName,
// 		DistributeBy:     order.DistributeBy,
// 		OrderDate:        order.OrderDate,
// 		ArrivalDate:      order.ArrivalDate,
// 		SendDate:         order.SendDate,
// 		IsReceived:       order.IsReceived,
// 		DemandCode:       order.DemandCode,
// 		Money:            order.Money,
// 	}
// }

// func ProtoConditionToModelCondition(c *pb.ListOrderRequest) *model.OrderCondition {
// 	odf, _ := ptypes.Timestamp(c.OrderDateFrom)
// 	odt, _ := ptypes.Timestamp(c.OrderDateTo)
// 	adf, _ := ptypes.Timestamp(c.ArrivalDateFrom)
// 	adt, _ := ptypes.Timestamp(c.ArrivalDateTo)
// 	sdf, _ := ptypes.Timestamp(c.SendDateFrom)
// 	sdt, _ := ptypes.Timestamp(c.SendDateTo)
// 	return &model.OrderCondition{
// 		ReceiveBy:       c.ReceiveBy,
// 		DistributeBy:    c.DistributeBy,
// 		OrderDateFrom:   odf,
// 		OrderDateTo:     odt,
// 		ArrivalDateFrom: adf,
// 		ArrivalDateTo:   adt,
// 		SendDateFrom:    sdf,
// 		SendDateTo:      sdt,
// 		IsReceived:      c.IsReceived,
// 		DemandId:        c.DemandId,
// 		ProductId:       c.ProductId,
// 		Limit:           c.Limit,
// 		Offset:          c.Offset,
// 	}
// }

func ProtoConfirmSendRequestsToModelConfirmSend(c *pb.ConfirmSendRequests) (result []*model.ConfirmSend) {
	result = make([]*model.ConfirmSend, len(c.ConfirmSends))
	for i, item := range c.ConfirmSends {
		s, _ := ptypes.Timestamp(item.SendDate)
		a, _ := ptypes.Timestamp(item.ArriveDate)
		cs := &model.ConfirmSend{
			Id:         item.Id,
			SendDate:   s,
			ArriveDate: a,
			Details:    item.Details,
			AdjustImg:  item.Img,
			Type:       item.Type,
		}
		result[i] = cs
	}
	return
}

func ProtoCancelSendRequestsToModelCancelSend(c *pb.CancelSendRequests) (result []*model.CancelSend) {
	result = make([]*model.CancelSend, len(c.Ids))
	for i, item := range c.Ids {
		cs := &model.CancelSend{
			Id:     item,
			Reason: c.Reason,
		}
		result[i] = cs
	}
	return
}

func ProtoConfirmSendRequestToModelConfirmSend(c *pb.ConfirmSendRequest) (result *model.ConfirmSend) {
	return &model.ConfirmSend{
		Id:        c.Id,
		Details:   c.Details,
		AdjustImg: c.Img,
	}
}

func getFromAndToTime(t []*timestamp.Timestamp) (from, to time.Time) {
	if len(t) == 2 {
		from, _ = ptypes.Timestamp(t[0])
		from = time.Date(from.Year(), from.Month(), from.Day(), 0, 0, 0, 0, from.Location())
		from = from.Add(-8 * time.Hour)
		to, _ = ptypes.Timestamp(t[1])
		to = time.Date(to.Year(), to.Month(), to.Day(), 0, 0, 0, 0, to.Location())
		to = to.Add(24 * time.Hour)
		to = to.Add(-8 * time.Hour)
	}
	return
}

func ProtoOrderRequestToModelOrderCondition(c *pb.ListOrderRequest) *model.OrderCondition {
	odf, odt := getFromAndToTime(c.OrderDate)
	adf, adt := getFromAndToTime(c.ArrivalDate)
	sdf, sdt := getFromAndToTime(c.SendDate)
	rdf, rdt := getFromAndToTime(c.ReturnDate)
	return &model.OrderCondition{
		StoreId:         c.StoreId,
		SupplyId:        c.SupplyId,
		Status:          c.Status,
		Type:            c.Type,
		DemandId:        c.DemandId,
		JdeCode:         c.JdeCode,
		HwsCode:         c.HwsCode,
		OriginDemandId:  c.OriginDemandId,
		ProductId:       c.ProductId,
		OrderDateFrom:   odf,
		OrderDateTo:     odt,
		ArrivalDateFrom: adf,
		ArrivalDateTo:   adt,
		SendDateFrom:    sdf,
		SendDateTo:      sdt,
		ReturnDateFrom:  rdf,
		ReturnDateTo:    rdt,
		Limit:           c.Limit,
		Offset:          c.Offset,
		SortBy:          c.SortBy,
		Desc:            c.Desc,
	}
}

func ModelOrdersToProtoOrders(ctx context.Context, os []*model.Order) (result []*pb.Order) {
	var storeIds []uint64
	var resp *entity.ListEntityResponse
	var err error
	tempMap := map[string]byte{}
	for _, item := range os {
		l := len(tempMap)
		tempMap[item.StoreId] = 0
		if len(tempMap) != l {
			storeId, err := strconv.ParseUint(item.StoreId, 10, 64)
			if err != nil {
				panic(err)
			}
			storeIds = append(storeIds, storeId)
		}
	}

	//todo：从oauth取用户信息
	ctx = metadata.AppendToOutgoingContext(ctx, "partner_id", strconv.FormatUint(4183192445833445399, 10), "scope_id", strconv.FormatUint(0, 10), "user_id", strconv.FormatUint(4428480697887358977, 10))

	queryRequest := &entity.ListEntityRequest{
		SchemaName:   "STORE",
		Ids:          storeIds,
		ReturnFields: "id,name",
		Relation:     "company_info",
		Limit:        -1,
	}
	if resp, err = entity.DefaultClient.ListEntity(ctx, queryRequest); err != nil {
		return
	}
	fmt.Println(resp)
	var companyIds []uint64
	storeCompanyMap := make(map[uint64]interface{})
	for _, item := range resp.Rows {
		if etContent, err := utils.Struct2Map(item.Fields); err == nil {
			companyId := utils.GetUint64ArrayFromJSON(utils.GetMapFromJSON(etContent, "relation"), "company_info")
			companyIds = append(companyIds, companyId...)
			storeCompanyMap[item.Id] = companyId[0]
		}
	}
	companyMap := make(map[uint64]map[string]interface{})
	if len(companyIds) > 0 {
		queryRequest = &entity.ListEntityRequest{
			SchemaName:   "COMPANY_INFO",
			Ids:          companyIds,
			ReturnFields: "id,code,name",
			Limit:        -1,
		}
		if resp, err = entity.DefaultClient.ListEntity(ctx, queryRequest); err != nil {
			return
		}
		for _, item := range resp.Rows {
			if etContent, err := utils.Struct2Map(item.Fields); err == nil {
				companyMap[item.Id] = etContent
				fmt.Println(etContent)
			}
		}
	}

	for _, item := range os {
		od := &pb.Order{
			Id:              item.Id,
			Code:            item.Code,
			JdeCode:         item.JdeCode,
			HwsCode:         item.HwsCode,
			Type:            item.Type,
			StoreId:         item.StoreId,
			SupplyId:        item.SupplyId,
			Status:          item.Status,
			Price:           item.Price,
			ProductId:       item.ProductId,
			Quantity:        item.Quantity,
			RealQuantity:    item.RealQuantity,
			RecevedNum:      item.RecevedNum,
			RealRecevedNum:  item.RealRecevedNum,
			UnitId:          item.UnitId,
			OriginOrderCode: item.OriginOrderCode,
			AdjustImg:       item.AdjustImg,
			TaxRate:         item.TaxRate,
			Reason:          item.Reason,
		}
		pid := cast.ToUint64(item.StoreId)
		companyId := storeCompanyMap[pid].(uint64)
		companyInfo := companyMap[companyId]
		od.CompanyId = companyId
		od.CompanyCode = companyInfo["code"].(string)
		od.CompanyName = companyInfo["name"].(string)

		if !item.OrderDate.IsZero() {
			od.OrderDate = item.OrderDate.Format(time.RFC3339)
		}
		if item.Type == "要货单" && item.Status != "INITED" && item.Status != "PENDING" && !item.ArrivalDate.IsZero() {
			od.ArrivalDate = item.ArrivalDate.Format(time.RFC3339)
		}
		if item.Type == "要货单" && item.Status != "INITED" && !item.SendDate.IsZero() {
			od.SendDate = item.SendDate.Format(time.RFC3339)
		}
		if !item.ReturnDate.IsZero() {
			od.ReturnDate = item.ReturnDate.Format(time.RFC3339)
		}
		if item.Type == "要货单" && !item.ExpectArrivalDate.IsZero() {
			od.ExpectArrivalDate = item.ExpectArrivalDate.Format(time.RFC3339)
		}
		if !item.UpdatedAt.IsZero() {
			od.UpdatedAt = item.UpdatedAt.Format(time.RFC3339)
		}
		//if item.UpdatedBy != 0 {
		//	od.UpdatedBy = oauth.GetUserName(ctx, item.UpdatedBy)
		//}
		result = append(result, od)
	}
	return
}

func ProtoConfirmEnsureRequestToModelConfirmEnsure(ceq *pb.ConfirmEnsureRequest) *model.ConfirmEnsure {
	return &model.ConfirmEnsure{
		Type: ceq.Type,
		Ids:  ceq.Ids,
	}
}

// func ModelOrdersToProtoOrdersDetails(orders []*model.OrderWithProductPrice) (result []*pb.OrderWithMoney) {
// 	for _, item := range orders {
// 		result = append(result, ModelOrderWithProductPriceToProtoOrder(item))
// 	}
// 	return
// }

// func ModelOrderWithProductPriceToProtoOrder(order *model.OrderWithProductPrice) *pb.OrderWithMoney {
// 	return &pb.OrderWithMoney{
// 		Id:               order.Id,
// 		Code:             order.Code,
// 		StoreSecondaryId: order.StoreSecondaryId,
// 		StoreName:        order.StoreName,
// 		DistributeBy:     order.DistributeBy,
// 		OrderDate:        order.OrderDate,
// 		ArrivalDate:      order.ArrivalDate,
// 		SendDate:         order.SendDate,
// 		IsReceived:       order.IsReceived,
// 		DemandCode:       order.DemandCode,
// 		ProductId:        order.ProductId,
// 		Quantity:         order.Quantity,
// 		UnitId:           order.UnitId,
// 		Money:            order.Price,
// 	}
// }

```

Api:http://seesaw-boh.hexcloud.cn/api/v1/supplier-platform/order/query

查找所调用方法：/proto/reveive/indent.proto下的SupplyIndentService.QueryOrder

更改proto：

```go
message Order {
    uint64 id                  = 1;     // 单据 id
    string code                = 2;     // 要货单编号
    string jde_code            = 3;     // JDE 单号
    string hws_code            = 4;     // HWS 单号
    string type                = 5;     // 单据类型
    string store_id            = 6;     // 门店编号
    uint64 supply_id           = 7;     // 供应商 id
    string order_date          = 8;     // 要货日期
    string arrival_date        = 9;     // 到货日期
    string send_date           = 10;    // 发货日期
    string return_date         = 11;    // 退货日期
    string status              = 12;    // 状态
    double price               = 13;    // 价格
    uint64 product_id          = 14;    // 商品 id
    double quantity            = 15;    // 商品数量
    double real_quantity       = 16;    // 实际商品数量
    uint64 unit_id             = 17;    // 商品单位
    string origin_order_code   = 18;    // 原单据编号
    double receved_num         = 19;    // 收货数量
    double real_receved_num    = 20;    // 实际收货数量
    string adjust_img          = 21;    // 调整单图片
    string expect_arrival_date = 22;    // 预估到货日期
    string updated_by          = 23;    // 更新人
    string updated_at          = 24;    // 更新时间
    double tax_rate            = 25;    // 税率
    string remark              = 26;    // 备注
    string withdraw_at         = 27;    // 测回时间
    string withdraw_reason     = 28;    // 撤回原因
    string reason              = 29;    // 驳回原因
    uint64 company_id          = 30;    // 公司id
    string company_code        = 31;    // 公司code
    string company_name        = 32;    // 公司名称
}
```

编译proto：

```
protoc  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \--go_out=plugins=grpc,paths=source_relative:. --validate_out="lang=go,paths=source_relative:." indent.proto
```

```
--go_out=Mgoogle/api/annotations.proto=github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis/google/api,plugins=grpc:.
```

 移动pkg/utils

代码结构：

* grpc：下的go文件，是方法入口， indent.go
* service：处理基本的逻辑， platform.go
* repo/database: 数据库逻辑处理， indent_platform.go



# 春节假期问题

深圳湾万象城，报损每天都会保存失败

门店的反馈，报损页面停留超过五分钟了再电脑上保存报损单，可能会提示报损失败，然后报损的都为0

失败日志：

umarshal error fail: Exception calling application: (raised as a result of Query-invoked autoflush; consider using a session.no_autoflush block if this flush is occurring prematurely) (MySQLdb._exceptions.IntegrityError) (1062, "Duplicate entry '4459220566650617857-4294354923163877376-01' for key 'supply_adjust_product_unique_index'") [SQL: 'INSERT INTO supply_adjust_product (created_at, updated_at, id, partner_id, user_id, adjust_id, product_id, product_code, product_name, unit_id, unit_name, unit_spec, accounting_unit_id, accounting_unit_name, accounting_unit_spec, quantity, accounting_quantity, confirmed_quantity, is_confirmed, item_number, material_number, adjust_store, adjust_date, reason_type, convert_accounting_quantity, is_bom) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)'] [parameters: ((datetime.datetime(2021, 2, 17, 0, 9, 14, 509406), datetime.datetime(2021, 2, 17, 0, 9, 14, 512504), 4459426603895717888, 4206763836826861569, 4231712376929796168, 4459220566650617857, 4294354923163877376, '30990058', '紫米紫薯米面包', 4219460847195389953, '个', 'EA', 4219460847195389953, '个', 'EA', 9.0, 9.0, None, 0, 1, '30990058', 4219460804656758785, datetime.datetime(2021, 2, 16, 0, 0), '01', None, 0), (datetime.datetime(2021, 2, 17, 0, 9, 14, 511622), datetime.datetime(2021, 2, 17, 0, 9, 14, 512511), 4459426603904139264, 4206763836826861569, 4231712376929796168, 4459220566650617857, 4294354923465768960, '30990059', '芋泥咸蛋黄米面包', 4219460847195389953, '个', 'EA', 4219460847195389953, '个', 'EA', 6.0, 6.0, None, 0, 2, '30990059', 4219460804656758785, datetime.datetime(2021, 2, 16, 0, 0), '01', None, 0))] (Background on this error at: http://sqlalche.me/e/gkpj)



# 销售配方物料拆解异常报表

## 测试数据

```sql
select e.pos_order_id,
e.sequence_id, 
pc."id" as product_id, 
pc.code as product_code, 
pc.name as product_name, 
sc."id" as store_id, 
sc.code as store_code, 
sc.name as store_name, 
b.error from (
select bom_explain_ret,
  e_ticket_id,
             bom_explain_ret::json #>> '{response,0,error}'      as error,
             bom_explain_ret::json #>> '{response,0,store_id}'   as store_id,
             bom_explain_ret::json #>> '{response,0,product_id}' as product_id
      from sales_inventory_detail
  where updated > 1609868607 and bom_explain_ret::json #>> '{response,0,error}' is not null
   and bom_explain_ret::json #>> '{response,0,error}' like '%no_bom%') b 
   inner JOIN product pc on pc."id" = b.product_id 
   inner join store sc on sc."id" = b.store_id
   inner join e_ticket e on e."id" = b.e_ticket_id
```

```
label:not_match_label_group;material:no_inventory:333327176646589183;material:no_inventory:333327177804217087;material:no_inventory:333327193105038079;material:no_inventory:333327245852605183;material:no_inventory:333327246607579903;material:no_inventory:333327254039886591;material:no_inventory:333327359686015743



material:no_inventory:333327176646589183;material:no_inventory:333327177804217087;material:no_inventory:333327193105038079;material:no_inventory:333327245852605183;material:no_inventory:333327246607579903;material:no_inventory:333327254039886591;material:no_inventory:333327359686015743



no_bom
```

```
{"response": [{"request_id": "4465088522830860289", "product_id": "4437203767837884416", "store_id": "4437198984162869248", "sales_date": "2021-03-04", "product_qty": 1.0, "error": "bom:no_bom"}, {"request_id": "4465088522830860289", "product_id": "4437202966243475456", "store_id": "4437198984162869248", "sales_date": "2021-03-04", "product_qty": 1.0, "error": "bom:no_bom"}]}


```





## 需求

> 维度：按门店按单按商品—详情到配方物料
>
> 异常类型1：商品无配方的情况（过滤零售物料后）——按门店按订单按品
> 异常原因：—列表新增字段
> ① 商品无对应配方—爆出商品，写入异常描述
> ② 门店与商品配方是没有对应关系—爆出商品和门店，写入异常描述
>
> 异常类型2：走兜底逻辑的拆解——按门店按单品维度
> 异常原因：
> ① 标签不匹配直接兜底--爆出原标签组和拆解的标签组，写入异常描述
> ② 标签重复去重后兜底—爆出重复标签、原标签组和拆解的标签组，写入异常描述
> ③ 同一可选项下有多个标签，丢弃一个后兜底-爆出同组标签、原标签组和拆解的标签组，写入异常描述
>
> 异常类型3：无库存不拆解——按门店按单按品按配方物料
> 异常原因：
> ① 配方物料下原物料无库存—暴出商品和配方物料，写入异常描述

## 开发

* 增加接口：/Users/shenshaoxuan/hekuo/report/report/api/operation_report.py----GetSalesExplainedErrorFromMetadataReport

## 修复慢sql，重新开发

测试

```python
def test108():
    with grpc.insecure_channel('localhost:8686') as channel:
        stub = operation_report_pb2_grpc.OperationReportStub(channel)
        request = GetSalesExplainedErrorReportRequest()

        start = datetime.now() - timedelta(days=30)
        end = datetime.now() - timedelta(days=35)
        timestamp = Timestamp()
        timestamp.FromDatetime(start)
        start_date = timestamp
        request.start_date.seconds = 1610132422
        # request.start_date.seconds = start_date.seconds
        timestamp = Timestamp()
        timestamp.FromDatetime(end)
        end_date = timestamp
        request.end_date.seconds = 1612638022
        #  request.end_date.seconds = end_date.seconds
        # request.error_type = 'no_inventory'
        # request.error_type = 'no_label'
        request.error_type = 'no_bom'
        request.limit = 10
        # request.offset = 100
        # request.product_ids.extend([4437202168889507840])
        # request.store_ids.extend([4437198985588932608])
        # request.pos_ticket_number = '50002'
        response = stub.GetSalesExplainedErrorFromMetadataReport(request, metadata=metadata)
    print(response)
```



* 修改bom_service_task:重写ret数据到表
* 新建model，
* 同步model到数据库，建立索引
* 生成sql以便升级
* 重写查询接口



# pos的user下发排序

* 后端已排序，需要林炳强确认给前端数据的排序方式

  

# 建议订货量

## 自动任务

### 建议预估初始化

##### 1. 接收参数

* partner_id
* user_id
* demand_date：将当天日期加一天，每日计算的应该是下日的
* batch_type：批次类型，必传

##### 2. 流程

* 创建SupplySuggestBatch
* 创建日期设为12天
* 查找所有门店信息(条件是open状态，并且可用的)
* 创建SupplySuggestStoreBatch，全部插入
* 循环SupplySuggestStoreBatch， 循环发送消息supply.suggest，开始预估

### 计算预估销量

* 一次处理一家门店，一天的预估数据

* 根据id查找第二层信息
* 锁住批次信息
* 从主档获取天气因子、节日因子、促销因子
* 查询销售记录表，无条件，只要有销售记录的商品都会查询
  * 查询前四周同比天的每件商品的毛营业额（净额+折扣额度）
  * 获取四周同比天的平均营业额
  * 获取每周同比天的营业额
  * 获取四周同比天每天销售的商品id
* 查询销售记录表，查询前天往前四天，按照商品分组，计算商品的销量数量，销售金额
  * 获取商品的每日平均销量和平均单价
  * 得到每日的平均真实销售金额

* 获取前四天的商品的每日平均报损数量
* 汇总四周全部的销量商品
* 计算系统预估金额 = 最近一周 *  0.5 + （剩余三周的和）* 0.5 
* 计算千元销售 = 商品近4天的平均销量 / 真实销售金额 * 1000
* 计算预估销量 = 千元销售 * 预估金额 / 1000
* 写入数据

```json
{
  demand_date
  product_id
  store_id
  sys_amount
  thousand_form
  sys_quantity
  adjust_qty
  real_amount
  price
}
```

### 计算建议订货量

**触发条件**: 销量预估更新完毕后，用户手动更改销售后

查询sales_explained_report_daily_build

* 获取商品id，bom_id，sum(商品数量)，sum(bom数量)
* 获取bom在每件商品的用量，得到dict===>>>>
* 获取商品在单位时段内的总销量

```
查询出需要计算建议订货量的商品
获取供应商安全系数
获取仓库安全系数
获取产品上周总销量
将面包从可订货商品中剥离出来

```

​			插入supply_suggest_product
​			更新订货量
​				查询门店供应商
​				orm商品
​				查询实时库存
​				获取在途商品
​					不同商品类型查询的时段是不一样的
​				查询出需要计算建议订货量的商品

查询出需要计算建议订货量的商品

获取在途商品

实时库存获取，如果配送类型是bread：加入oem_ids

## 新增接口

#### 查询预估列表

* 订货周期写死11天，查询这11天的所有系统预估数据(~~原先查询方式是根据当天的订货单查询出的商品获取一个最长的订货周期~~)
* 获取新上架商品详情(在销售记录表中查询与昨日相比，新增的销售商品，条件为：成品，并且distinct去重(有可能存在退单的商品单价为负数))
* 获取一个每日的基础数据，比如real_amount，sys_amount，reason，后面需要在每条数据上进行填充
* 根据周期，循环预估数据，如果商品在这个预估周期每日的千元结构均为0，并且不属于新商品，并且预估数据也全为0，则删除不显示，否则认为是新商品
* 处理新商品数据，如果不在以上预估数据中，则加入到列表
* 查询当天订货单是否审核，返回数据可更新状态

#### 更新预估数据接口

* 校验数据，如果存在已经审核的订货单(根据时间确定查询日期，13点前后不一样的条件)，直接抛错
* 整合用户数据，将数据按照天分类，分类中保存着金额数据和销量预估数据
* 校验用户修改的日期，不能修改当日之前数据
* 查询近两日的销售数据的商品单价
* 按照日期取出所有的预估数据
* 循环预估数据 
  * 如果只修改了金额（修改过的商品销量，默认被锁定）
    * 如果商品修改过销量，则刨除金额，否则保存商品的千元结构和单价
    * 则预估金额的计算公式为：预估金额 = 金额/sum(商品的千元销售*单价)
    * 更新数据，被锁定的和新上架的只需要更新预估金额，而非新上架的需要更新预估金额，并重算预估数量
  * 如果只修改了销量
    * 改插入的插入，改更新的更新，统计总金额应该加多少
  * 如果同时修改
    * 则预估金额的计算公式为：预估金额 = 金额/sum(商品的千元销售*单价)

#### 获取门店最后的更新信息

查询预估更新日志，返回更新人，操作日期

#### 一键回退预估数据

当天数据不允许修改，直接将近10天的数据回归到最初的系统预估状态，清空两个修改状态标识，两个reason，两个pre


## 插入新商品做测试

中山白水井店

```sql
-- 查询当前门店还没写入到预估表的商品
select *
from metadata_product
where id not in (select product_id from supply_suggest where store_id=4413991449748668416 and demand_date='2021-04-24')
and product_type='FINISHED'
```

插入一些商品到销售表

```sql
-- 获取当天日期
select to_timestamp(1615824000)
INSERT into product_sales_record(id, partner_id, store_id, operation_date, product_id, price)
VALUES(4469499136081657841, 4206763836826861569, 4413991449748668416, 1619107200, 4219461151768969217, 101)
```

## 验证预估商品的数量

```sql
-- 今日触发任务时，查询过去四周同比天的销售数据，所有销售商品数据应该与预估表当天的预估商品数量一致
SELECT product_id FROM product_sales_report
WHERE operation_date=1615478400 AND store_id=4219460796045852673  GROUP BY store_id,product_id
union 
SELECT product_id FROM product_sales_report
WHERE operation_date=1614873600 AND store_id=4219460796045852673  GROUP BY store_id,product_id
union
SELECT product_id FROM product_sales_report
WHERE operation_date=1614268800 AND store_id=4219460796045852673  GROUP BY store_id,product_id
union
SELECT product_id FROM product_sales_report
WHERE operation_date=1613664000 AND store_id=4219460796045852673  GROUP BY store_id,product_id

-- 查询预估表当天实际写入的商品数量
select product_id from supply_suggest where store_id=4219460796045852673 and demand_date='2021-03-19' order by product_id
```

## 锁定功能

### 单元测试

* 查询接口
  * 已审核报错
  * 12点不能修改当天报错
  * 普通查询
* 保存接口
  * 只修改金额
  * 只修改销量
  * 同时修改
  * 写一个测试接口在每种类型保存后验证销量乘以单价是否与金额一致

### 原设计

> 用户更改了预估金额 ===>>>>系统根据用户更改的金额使用千元销售反推各个商品的销量
>
> 然后更改A商品的销售数量 ====>>>>>>系统根据用户更改的数量与第一步修改金额后得到的数量的差量 * 单价，计算出总金额追加到原预估金额上
>
> 如果用户此时点击保存：
>
> ​	则前端必须将预估金额与A商品的销量传递给后端,
>
> ​	后端做法为：
>
> ​			1. 待分摊金额 =（更改后的预估金额-A商品的数量*单价）
>
> ​			2. 使用待分摊金额 ，根据其余商品的千元销售反推其余商品的销量	
>
> 如果用户不保存，继续修改预估金额===>>>>系统根据用户更改的金额使用千元销售反推各个商品的销量, 则刚修改的A商品的销量会被覆盖
>
> 
>
> 因此，原先设计为用户不能同时修改金额与商品销售数量



注：

> 修改金额时：如果发现商品已全部被锁定，则提醒用户此时金额无法修改(因为此时没有商品能分摊新增或减少的预估金额)
>
> 修改销量时：如果发现只剩余一件未锁定商品，并且金额也被锁定的情况下，提醒用户无法修改此商品销量(因为此时没有商品能分摊新增或减少的预估金额)



### 后台保存接口

如果只修改了金额：

* 修改过销量的商品默认被锁定，金额分摊到其余商品

如果只修改了销量：

* 修改后的销量与修改前的销量的数量差 * 单价 新增到总金额上

如果同时修改了金额和销量

* 修改过销量的商品与本次提交更改了销量的商品被锁定，金额分摊到其余商品



## 新的建议订货量规则

商品销售记录中保存了销售过的商品的每个标签

例如拿到芝芝芒芒的所有标签组合请求bom，得到所有的bom配方

例如

```json
{
  "标签1": {
    "吕燕": {
      "吕燕1号": 50,
      "吕燕2号": 60,
      "吕燕3号": 70
    },
    "牛奶": {
      "牛奶1号": 150,
      "牛奶2号": 160
    }
  },
  "标签2": {
    "吕燕": {
      "吕燕1号": 50,
      "吕燕2号": 60,
      "吕燕3号": 70
    },
    "矿泉水": {
      "矿泉水1号": 150,
      "矿泉水2号": 160
    }
  }
} 
```

做一个dict，bom_id作为key，qty作为value

循环每个标签

将订单中的原料做成一个dict

循环标签中的每一个物料，如果物料存在则使用该物料，将标签的数量*物料用量累加到dict中



# 喜茶新增接口

1. 合阔云端服务提供openAPI   （GET，oauth认证，固定token)    （喜茶希望自己拉取版本信息）
2、version list：喜茶提供版本信息，合阔云端服务提供ADD接口，将信息写入到合阔的信息表中（consul) 

3、在线列表版本上报：合阔云端服务提供接口 上报目前该门店的使用版本号[ 喜茶提供门店第二编号,类似我们POS 机的在线列表的信息】



1. 参数,store_id, app_name
   * 主档获取当前版本===>>>version
2. 参数：app_name， version， oss地址
   * 写到consul




# pos菜单不一致

取缓存

```
hex:4206763836826861569:0:pos_kb_keys:4353130198439297025
```



通过pos找到门店，查看门店的自定义菜单

然后修改这个菜单，保存

查看菜单缓存是否有改动，来确定保存后是否build缓存

如果有改变，则查看是哪个位置build了缓存



经查看，是通过自动任务创建的缓存

* 不能通过判断section_id=0，来过滤数据，因为某个大类自动带入的数据确实是没有section_id和section_order
* 如果用户没有保存过，则section_order和section_id全为0，不存在只有一个的情况

经过排查：

确定唯一的解决办法是修改保存接口，将数据清除掉

查找section_id和section_order为何会为0

前端传入参数：

​	section_id：1234

​	parent_id：123

后台：

​	如果当前页面已经保存过商品，则条件为delete(keyboard=123, section_id=123, id not in (ids), scope_id=1)

​	如果用户没保存过，直接通过商品大类带入的，则以上这种删除方式无法删除

​	新做法：

​		delete(keyboard=123,, id not in (ids),id in (parent_id.childs), section_Id<>0  , scope_id=1)---->>>这种情况可能会删除掉用户已经在其他页保存的数据

​		删除掉数据，并且不能影响当前parent_id下的已经保存的数据



例如1：删除未保存的第二页的A商品，如果第一页用户保存过A商品，则不能影响第一页

​	第一页的A商品一定是有坐标数据和section_id的



例如2：删除已保存的第二页的A商品，如果第一页用户保存过A商品，则不能影响第一页

​	第一页的A商品一定是有坐标数据和section_id的





如何让section_id和section_order变为0

问题：

门店id：

keyboard_id：4434642458827063296



------

# 熔断API



> Issue:https://www.tapd.cn/37647819/prong/tasks/view/1137647819001003004

Hex_server_api

## 创建表

```sql
create table if not exists bills_fusing_log
(
	id bigint not nuinglll
		primary key,
	store_id bigint default 0 not null,
	store_code varchar(50) default '' null,
	pos_id bigint default 0 not null,
	pos_code varchar(50) default '' null,
	close_type int default 0 not null comment '1.熔断；2.关闭',
	close_time datetime null,
	close_date date null,
	reason varchar(200) null,
	created datetime null
)
comment '插单熔断日志';

create index bills_fusing_log_pos_id_index
	on bills_fusing_log (pos_id);

create index bills_fusing_log_store_id_index
	on bills_fusing_log (store_id);

```

## 分支

feature/20210408_add_fusing_api



# pos-sku排序

pos下发打印顺序或者排序好后下发 

* Keyboard_main:排序
* products：排序

商品附加属性无论怎么调整，顺序都按照打印顺序显示

背景同步：由于HIPOS后台属性，目前系统是按照我们勾选的先后展示的pos端，造成每一个商品展示的顺序不同，店员在操作的时候会显示有点乱，有的温度在前，有的冰度在前，打印出来的大标签顺序也可以进行排序

需求：针对商品属性(加料，状态，冰量，甜度，茶底，做法，口味，分装.....)POS后台配置，客户端UI展示，大标签打印三者的顺序都可以进行自定义排序。

1、对分类和属性，可以进行排序的显示，可以让所有的商品都统一按照指定的顺序显示，比如：我们发现pos展示的属性组及属性前后顺序，是按照设置时候勾选的先后顺序展示，这样会出现每个商品的属性组展示的前后顺序都不一样，门店看着也很乱，可以优化成固定的顺序吗？(状态，冰量，甜度，茶底，做法，口味，分装.....)

2、UI展示的商品属性，排序需要规范统一的排序，目前每个商品展示的冰度糖度都不相同

3、标签打印顺序，加料也可以排序，目前加料无法排序



# MRP订货明细报表

* 订单满足销售区间
  * 得到原料所属单据的订货类型
  * 得到供应商的配置项，算出到货日期
  * 则结果为订单日期---到货日期
* 原料编号，名称，商品编号，名称
* 预估产品销量
  * suggest去sum订货周期内的，这个商品，这个门店的系统预估销量
* 实际产品销售量
  * 门店销售报表这个周期内的商品的实际总销量
* 手动修改产品销量
  * 用户修改的这个周期内sum
* BOM配方：平均每杯的平均耗量
* 预估使用量(预估单位)：suggest
* 实际使用量(实际单位)
* 预估销量VS实际销量准确率：可能是不准确的数字
* 手工修改销量vs实际销量准确率：可能是不准确的数字
* 报损：adjust有商品的每日平均报损，未销售则为0



# hipos商品重叠

分支：feature/20210419_resolve_keyboard_overlap

解决商品重叠问题

保存时，如果是通过商品带入的，自动为每个key安排页码



# hipos删除空白页

分支：feature/20210419_resolve_keyboard_overlap

# bom物料替换更改

* 更改proto，请求新增标识is_suggest

```go
message BatchRequest {
             uint64            request_id = 1; // 请求id 唯一
             uint64            store_id   = 2; // 门店id
             string            sales_date = 3; // 营业日
    repeated ProductBomReqeust products   = 4; // 商品明细
             bool              is_suggest = 5; //是否是建议订货量分解请求
}
```

* response新增字段id，便于supply对原料分组

```go
message TProdoctBom {
    uint64 product_id = 1; // 原料id
    uint64 uom_id     = 2; // 单位（配方）
    double qty        = 3; // 数量
   uint64 material_id = 4; // 物料id
}
```



如果is_suggest为真则在请求库存的这一步做一个分支

* 不用请求库存？
* 数据库获取到所有原料详情，则按照转换比率转换
* 返回response

```json
{
  333327151380101887(绿研):[
    {
      "product_id":4437200541482778624,
      "rate":1
    },
    {
      "product_id":4437200541616996352,
      "rate":1.1
    },
  ],
   333327151380101888(奶油):[
    {
      "product_id":4437200541482778622,
      "rate":1
    },
    {
      "product_id":4437200541616996351,
      "rate":1.1
    },
  ]
}
```

原料的单位使用物料的单位



## redis服务器迁移

keyboard可以从缓存或者数据库中获取



registry.hexcloud.cn/histore/supplier-platform/seesaw-master:b02cf70c

registry.hexcloud.cn/histore/supplier-platform/seesaw-master:b02cf70c







registry.cn-shanghai.aliyuncs.com/hexcloud/histore/grpc-gateway/seesaw-master:14220960

测试：

registry.hexcloud.cn/histore/grpc-gateway/master:fc051754