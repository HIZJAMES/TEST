search
void CUberEquip::Draw(const r3dSkeleton* skel, const D3DXMATRIX& CharMat, bool draw_weapon, DrawType dt, bool first_person)

copy paste
void CUberEquip::Draw(const r3dSkeleton* skel, const D3DXMATRIX& CharMat, bool draw_weapon, DrawType dt, bool first_person)
{
    //todo: call extern void r3dMeshSetWorldMatrix(const D3DXMATRIX& world)
    // instead of mesh->SetWorldMatrix

    // in first person mode we need to render player and gun into different Z range
    if(dt == DT_AURA)
    {
        float expandConst[ 4 ] = { r_aura_extrude->GetFloat(), 0.f, 0.f, 0.f } ;
        D3D_V(r3dRenderer->pd3ddev->SetVertexShaderConstantF(23, expandConst, 1)) ;
    }

    skel->SetShaderConstants();

	for(int i=0; i<=SLOT_Backpack; i++)
	{
		if(d_disable_backpacks_draw->GetBool() && i==SLOT_Backpack)
            continue;

        DrawSlot((ESlot)i, CharMat, dt, true, first_person, NULL);
    }

    if(dt != DT_AURA)
    {
        D3DXMATRIX world = getWeaponBone(skel, CharMat);
        if(!first_person)
        {
            if(slots_[SLOT_WeaponBackRight].wpn)
            {
                world = getWeaponBone(skel, CharMat);
                bool skinned = false;
                r3dSkeleton* wpnSkel = game_new r3dSkeleton(); 
                Weapon* wpn = slots_[SLOT_WeaponBackRight].wpn;

                wpn->checkForSkeleton();

                if(wpn)
                {
                    r3dMesh* msh = wpn->getModel(true, first_person);
                    if(msh->IsSkeletal() && wpn->getConfig()->getSkeleton())
                    {
                        wpn->getAnimation()->Recalc();
                        wpn->getAnimation()->pSkeleton->SetShaderConstants();
                        wpnSkel = wpn->getAnimation()->pSkeleton;
                        skinned = true;
                    }                
                }
                
                skel->GetBoneWorldTM("Weapon_BackRight", &world, CharMat);
                DrawSlot(SLOT_WeaponBackRight, world, dt, skinned, first_person, wpnSkel);
            }

            if(slots_[SLOT_WeaponSide].wpn)
            {
                if(slots_[SLOT_WeaponSide].wpn->getCategory() == storecat_MELEE) {
                    skel->GetBoneWorldTM("Weapon_Side", &world, CharMat);

                    D3DXMATRIX mr1;
                    D3DXMatrixRotationYawPitchRoll(&mr1, 0, R3D_PI/2 + 40, 30);
                    skel->GetBoneWorldTM("Weapon_Side", &world, CharMat);
                    world = mr1 * world;

                    DrawSlot(SLOT_WeaponSide, world, dt, false, first_person, NULL);
                }
                else {
                    world = getWeaponBone(skel, CharMat);
                    bool skinned = false;
                    r3dSkeleton* wpnSkel = game_new r3dSkeleton();
                    Weapon* wpn = slots_[SLOT_WeaponSide].wpn;

                    wpn->checkForSkeleton();

                    if(wpn)
                    {
                        r3dMesh* msh = wpn->getModel(true, first_person);
                        if(msh->IsSkeletal() && wpn->getConfig()->getSkeleton())
                        {
                            wpn->getAnimation()->Recalc();
                            wpn->getAnimation()->pSkeleton->SetShaderConstants();
                            wpnSkel = wpn->getAnimation()->pSkeleton;
                            skinned = true;
                        }
                    }
					
                    skel->GetBoneWorldTM("Weapon_Side", &world, CharMat);
                    DrawSlot(SLOT_WeaponSide, world, dt, skinned, first_person, wpnSkel);
                }
            }
        }

        if(draw_weapon)
        {
            world = getWeaponBone(skel, CharMat);
            bool skinned = false;
            r3dSkeleton* wpnSkel = game_new r3dSkeleton();
            Weapon* wpn = slots_[SLOT_Weapon].wpn;
            if(wpn)
            {
                r3dMesh* msh = wpn->getModel(true, first_person);
                if(msh->IsSkeletal() && wpn->getConfig()->getSkeleton())
                {
                    wpn->getAnimation()->Recalc();
                    wpn->getAnimation()->pSkeleton->SetShaderConstants();
                    wpnSkel = wpn->getAnimation()->pSkeleton;
                    skinned = true;
                }
            }
            DrawSlot(SLOT_Weapon, world, dt, skinned, first_person, wpnSkel);
        }
    }
}

search 
r3dMesh* WeaponAttachmentConfig::getMesh( bool allow_async_loading, bool aim_model) const

copy paste
r3dMesh* WeaponAttachmentConfig::getMesh( bool allow_async_loading, bool aim_model) const
{
    if(m_type == WPN_ATTM_RECEIVER || m_type == WPN_ATTM_STOCK || m_type == WPN_ATTM_BARREL) // stats only attachments
        return NULL;

	if(m_itemID == 400136 || m_itemID == 400141 || m_itemID == 400142 || m_itemID == 400152)
		return NULL;


    if(!aim_model)
    {
        if(m_Model == 0)
        {
            //r3dOutToLog("WeaponAttachmentConfig->%s\n", m_ModelPath);
            m_Model = r3dGOBAddMesh(m_ModelPath, true, false, allow_async_loading, true);
            if(m_Model==0)
            {
                r3dError("ART: failed to load mesh '%s'\n", m_ModelPath);
            }
            r3d_assert(m_Model);
        }
        return m_Model;
    }
    else if(aim_model && m_type == WPN_ATTM_UPPER_RAIL)
    {
        if(m_Model_AIM == 0)
        {
            char aim_model[512]; 
            r3dscpy(aim_model, m_ModelPath);
            int len = strlen(aim_model);
            r3dscpy(&aim_model[len-4], "_AIM.sco");


            m_Model_AIM = r3dGOBAddMesh(aim_model, true, false, allow_async_loading, true);
            if(m_Model_AIM==0)
            {
                r3dError("ART: failed to load mesh '%s'\n", aim_model);
            }
            r3d_assert(m_Model_AIM);
        }
        if(g_camera_mode->GetInt()==2)
        {
            return m_Model_AIM;
        }
        else {
            return m_Model;
        }
    }


    return NULL;
}

