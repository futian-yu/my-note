**mybatis之pageHelper插件**



​       Mybatis的pageHelper分页插件，基本实现了分页，但他只对他后面的第一个查询实现分页。我们一般会把查询出来的list作为PageInfo类构造函数的入参，让PageInfo去获取分页相关的更多信息（如当前页数、页大小、当前页面第一个、最后一个元素在数据库中的行号、总行数、是否是第一页、是否是最后一页等）。这样前端直接拿到值就可以使用，就不用经过一系列的计算判断来处理页码的显示。

```java
@RequestMapping("/query")
	public String AdminQuery(@RequestParam(required = false,defaultValue = "1",value = "pn")Integer pn,ModelMap map,String adminId,String adminName,HttpServletRequest req){

		
			//这里采用总子账号的权限，根据admin的公司实现，本公司的账号只能看到本公司的管理员账号
			Admin currentAdmin=(Admin) req.getSession().getAttribute("admin");
			Admin admin = new Admin();
			admin.setAdminId(adminId);
			admin.setAdminName(adminName);
			admin.setAdminCompany(currentAdmin.getAdminCompany());
			
			//========下面三句才是分页查询的关键=========
			
			//引入分页查询，使用PageHelper分页功能
	        //在查询之前传入当前页，然后多少记录
	        PageHelper.startPage(pn,10);
	        //startPage后紧跟的这个查询就是分页查询
			List<Admin> adminList = adminService.selectAllAdmin(admin);
			//使用PageInfo包装查询结果，只需要将pageInfo交给页面就可以
	        PageInfo pageInfo = new PageInfo<>(adminList,10);
	        //pageINfo封装了分页的详细信息，也可以指定连续显示的页数
	        map.put("pageInfo", pageInfo);
			
			System.out.println("查询管理员数目"+ adminList.size());
		
			return "admin/queryAdmin";
	}
```

