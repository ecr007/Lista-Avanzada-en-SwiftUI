# Lista Avanzada en SwiftUI

```swift
import Foundation
import SwiftUI

struct NodeOutlineGroup<Node>: View where Node: Hashable, Node: Identifiable, Node: CustomStringConvertible{
    let node: Node
    let childKeyPath: KeyPath<Node, [Node]?>
    
    @Binding var mCenter:Int
    
    @State private var isExpanded: Bool = true
    
    @State private var toggleEmail:Bool = false
    @State private var toggleNoty:Bool = false
    
    @State private var record:NotyModel = NotyModel()
    
    // Se utiliza para que la primera vez no envie el request al servidor
    @State private var run:Bool = false
    @State private var isParent:Bool = false
    
    var body: some View {
        if node[keyPath: childKeyPath] != nil {
            DisclosureGroup(
                isExpanded: $isExpanded.onUpdate {
                    record.show = isExpanded
                    isParent = true
                    update()
                },
                content: {
                    if isExpanded {
                        ForEach(node[keyPath: childKeyPath]!) { childNode in
                            NodeOutlineGroup(
                                node: childNode,
                                childKeyPath: childKeyPath,
                                mCenter: $mCenter
                            )
                        }
                    }
                },
                label: {
                    // Print Parent
                    Text(record.name)
                }
            )
            .onAppear{
                record = node as! NotyModel
                
                // Set hide or show
                isExpanded = record.show
                run = true
            }
        }
        else {
            // Print Child
            HStack(alignment: .center){
                VStack(alignment: .leading, spacing: 0){
                    Text(record.name)
                    
                    Text(record.description)
                        .foregroundColor(Color("DISSTextColor"))
                        .font(.caption)
                        .fontWeight(.light)
                        .frame(alignment: .leading)
                }
                
                Spacer()
                
                Toggle(isOn: $toggleEmail.onUpdate{
                    record.statusEmail = toggleEmail
                    isParent = false
                    update()
                }) {
                    Text("Email")
                }
                .labelsHidden()
                .padding(.trailing,20)
                
                Toggle(isOn: $toggleNoty.onUpdate {
                    record.statusMobile = toggleNoty
                    isParent = false
                    update()
                }) {
                    Text("Notification")
                }
                .labelsHidden()
            }
            .onAppear{
                record = node as! NotyModel
                
                // Set status of toggle
                toggleNoty = record.statusMobile
                toggleEmail = record.statusEmail
                run = true
            }
        }
    }// End body
    
    func update(){
        if run{
            NotificationRequest().update(
                user: UserModel.instance()!,
                center_id: mCenter,
                notyModel: record,
                isParent: isParent
            ) { _ in
                
                //print("DEBUG: Response => \(response)")
            }
        }
    }
}
```

## USO

```swift
List(records){ record in
      NodeOutlineGroup(
          node: record,
          childKeyPath: \.children,
          mCenter: $mCenter
      )
  }
  .listStyle(PlainListStyle())
  .overlay(
      ProgressView()
          .opacity(showLoading ? 1 : 0)
  )
```
